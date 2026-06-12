# TUI-Test-Harness Hints

When the System Under Test is a Terminal User Interface (TUI), the step
harness must talk to a real PTY. A pipe is not enough: most TUIs (ratatui,
tui-rs, textual via Python, bubbletea, …) call into a TTY library
(crossterm, termion, …) on startup, panic if they detect "no TTY", and
never reach their error-handling code.

This file is a **reference, not a template**. Pull from it when the
project under test is a TUI. For non-TUI SUTs, ignore this file entirely.

## Why a PTY is unavoidable

- A pipe (`stdin`/`stdout` connected to a pipe) looks like "no TTY" to
  the child. `isatty()` returns false. crossterm, termion, and similar
  libraries either panic or hang in this state.
- A PTY is bidirectional, exposes a real `isatty()` to the child, and
  lets the TUI render its frames normally.

## Crates that work

| Crate | Notes |
|---|---|
| `portable-pty` (Rust) | Cross-platform PTY. Default choice. |
| `pty` (Rust, unix-only) | Simpler, faster, no Windows support. |
| `expectrl` (Rust) | Higher-level: spawn + expect + interact. |
| `pexpect` (Python) | Classic. `pexpect.spawn()`, `expect()`, `sendline()`. |
| `pty4j` (Java) | Bundled with some IDEs; standard JVM choice. |
| `creack/pty` (Go) | Standard Go PTY library. |

## Required harness capabilities

A working TUI harness must provide:

1. **Spawn** the binary attached to a PTY of a known size
   (e.g. 100×30 columns/rows). Set `TERM=xterm-256color` (or
   `xterm` if the binary doesn't recognize 256color).
2. **Send keys** as if from a real keyboard. Use the byte sequences the
   TUI library expects:
   - printable characters: their UTF-8 encoding
   - `Enter`: `\r`
   - `Esc`: `\x1b`
   - arrow keys: `\x1b[A` (Up), `\x1b[B` (Down), `\x1b[C` (Right), `\x1b[D` (Left)
   - `Page Up`/`Page Down`: `\x1b[5~`, `\x1b[6~`
3. **Capture the screen** as plain text rows. There are two paths:
   - **Parse the PTY output** through a VT/ANSI parser (`vte` in Rust)
     into a virtual character grid, then snapshot the grid. This is the
     only reliable way — the binary's actual rendered frame is what
     matters.
   - **Use the TUI's own `TestBackend`** (ratatui has one). Faster, no
     PTY, but couples the test to the TUI library and skips the
     keyboard-translation path. Use only when the SUT exposes a
     `pub fn run_with_backend(...)` for tests.
4. **Wait for stability** before snapshotting. Async loaders cause the
   screen to change over multiple frames. Naive `sleep` is flaky. Use
   one of:
   - Wait until the screen stops changing for N ms.
   - Wait until a specific marker appears (e.g. the commit list
     contains a row, the loading indicator disappears).
   - Wait until the panel you care about has the content you expect.
5. **Terminate** the child cleanly (`SIGTERM` / `kill` / `Child.kill()`).
   Then drain the reader thread (closing the PTY master does this).

## Stability anti-patterns

- `sleep(2.seconds())` then snapshot. **Flaky** — 2 s is sometimes not
  enough on slow CI, sometimes wasteful on fast machines.
- Snapshot the first frame. **Wrong** — the first frame is usually a
  half-rendered state with one of three panels still empty.
- Wait for a fixed number of redraws. **Wrong** — there's no fixed
  count; loaders complete asynchronously.

Recommended: a "ready" predicate per scenario, e.g. *"the Commits panel
contains a row whose first 7 chars are hex"*, with a poll loop and
timeout. If the predicate never holds, fail with a useful diagnostic
(the last frame at timeout).

## UTF-8 and box-drawing characters

TUI libraries emit Unicode box-drawing characters (`│`, `─`, `┌`, `┐`,
`└`, `┘`, `├`, `┤`) and selection markers (`> `, `  `). String slicing
in Rust panics on non-`char` boundaries if you index by raw bytes.

When parsing frames:

- Walk by `char`, not by byte. `row.chars()` is your friend.
- A "commit row" is `<decoration><7 hex chars><whitespace><subject>`,
  where `<decoration>` is one or more of `│`, `>`, ` `. Strip these
  before checking the hash.
- Panel borders are rows that start with `┌`, `─`, `│`, or `└`. The
  region between two `└` rows is a panel.

## Worked example: minimal ratatui app under test

A minimal harness for a Rust binary that uses ratatui + crossterm:

```rust
use portable_pty::{native_pty_system, CommandBuilder, PtySize};
use std::io::Read;
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::{Duration, Instant};
use vte::{Params, Perform};

// 1. State: a 100x30 character grid.
const ROWS: usize = 30;
const COLS: usize = 100;
#[derive(Default)]
struct Screen { rows: Vec<Vec<char>>, cursor_row: usize, cursor_col: usize }

// 2. Performer: implement vte::Perform to update the grid.
struct Performer<'a>(&'a mut Screen);
impl<'a> Perform for Performer<'a> {
    fn print(&mut self, c: char) { /* put char at cursor, advance */ }
    fn execute(&mut self, byte: u8) { /* handle \n, \r, \t, 0x08 */ }
    fn csi_dispatch(&mut self, params: &Params, _: &[u8], _: bool, action: char) {
        match action {
            'm' => { /* SGR — colors, mostly ignore for text checks */ }
            'H' | 'f' => { /* cursor position */ }
            'J' => { /* erase — mode 2 = clear screen */ }
            _ => {}
        }
    }
    fn hook(&mut self, _: &Params, _: &[u8], _: bool, _: char) {}
    fn put(&mut self, _: u8) {}
    fn unhook(&mut self) {}
    fn osc_dispatch(&mut self, _: &[&[u8]], _: bool) {}
    fn esc_dispatch(&mut self, _: &[u8], _: bool, _: u8) {}
}

// 3. Spawn.
pub struct Sut {
    writer: Box<dyn std::io::Write + Send>,
    state: Arc<Mutex<Screen>>,
    killer: Box<dyn portable_pty::ChildKiller + Send + Sync>,
}

pub fn spawn(bin: &std::path::Path, cwd: &std::path::Path) -> Sut {
    let pty = native_pty_system().openpty(PtySize { rows: 30, cols: 100, pixel_width: 0, pixel_height: 0 }).unwrap();
    let mut cmd = CommandBuilder::new(bin);
    cmd.cwd(cwd);
    cmd.env("TERM", "xterm-256color");
    let mut child = pty.slave.spawn_command(cmd).unwrap();
    let killer = child.clone_killer();
    let writer = pty.master.take_writer().unwrap();
    let state = Arc::new(Mutex::new(Screen::default()));
    let mut reader = pty.master.try_clone_reader().unwrap();
    let state_r = state.clone();
    thread::spawn(move || {
        let mut buf = [0u8; 4096];
        let mut parser = vte::Parser::new();
        loop {
            match reader.read(&mut buf) {
                Ok(0) | Err(_) => break,
                Ok(n) => {
                    let mut s = state_r.lock().unwrap();
                    let mut p = Performer(&mut s);
                    for &b in &buf[..n] { parser.advance(&mut p, b); }
                }
            }
        }
    });
    Sut { writer, state, killer }
}

// 4. Send keys.
pub fn send_key(sut: &mut Sut, key: Key) {
    use std::io::Write;
    let bytes: &[u8] = match key {
        Key::Char(c) => { let mut b = [0u8; 4]; let s = c.encode_utf8(&mut b); sut.writer.write_all(s.as_bytes()).unwrap(); return; }
        Key::Enter => b"\r",
        Key::Esc => b"\x1b",
        Key::Up => b"\x1b[A",
        Key::Down => b"\x1b[B",
    };
    sut.writer.write_all(bytes).unwrap();
    sut.writer.flush().unwrap();
}

// 5. Snapshot.
pub fn snapshot(sut: &Sut) -> Vec<String> {
    let s = sut.state.lock().unwrap();
    s.rows.iter().map(|r| r.iter().collect::<String>().trim_end().to_string()).collect()
}

// 6. Wait for stability.
pub fn wait_for_stable(sut: &Sut, stable_for: Duration, timeout: Duration) -> Vec<String> {
    let deadline = Instant::now() + timeout;
    let mut last = snapshot(sut);
    let mut last_change = Instant::now();
    loop {
        thread::sleep(Duration::from_millis(40));
        let now = snapshot(sut);
        if now != last { last = now.clone(); last_change = Instant::now(); }
        else if last_change.elapsed() >= stable_for { return now; }
        if Instant::now() >= deadline { return last; }
    }
}
```

## When NOT to use a PTY

- The SUT is a pure library with a `pub fn` that returns a state struct.
  Drive it directly from the test. No PTY needed.
- The SUT has a `pub fn run_with_test_backend(terminal, ...)` (ratatui
  provides this). Faster than a PTY, but skip the keyboard-translation
  path. Use only when the keyboard path is already covered by other
  tests.
- The SUT is a CLI tool with no TUI at all. Use plain
  `std::process::Command` or your language's equivalent.

## See also

- `cucumber-rust.md` — how to wire this into a `cucumber` test runner.
- The CAC skill's `cucumber-template.md` — index of framework templates.
