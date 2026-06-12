# Rust — `cucumber` crate

## Dependencies

```toml
# Cargo.toml
[dev-dependencies]
cucumber = "0.21"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
# for PTY-based TUI tests:
portable-pty = "0.8"
tempfile = "3"
```

## `[[test]]` declaration

```toml
[[test]]
name = "cucumber"
harness = false    # lets cucumber print its own output
```

## `tests/cucumber.rs`

```rust
// Test runner for CAC features.

use cucumber::World as _;

#[path = "../feature-tests/world.rs"]
mod world;

#[path = "../feature-tests/steps/mod.rs"]
mod steps;

#[tokio::main]
async fn main() {
    world::World::run("feature-tests").await;
}
```

## `feature-tests/world.rs`

```rust
use std::path::PathBuf;
use tempfile::TempDir;

#[derive(Default, cucumber::World, Debug)]
pub struct World {
    pub repo: Option<TempDir>,
    pub repo_path: Option<PathBuf>,
    pub output: Output,
    pub exit_code: Option<i32>,
    pub sut: Option<Sut>,
    pub last_frame: Vec<String>,
    pub active_panel: Option<String>,
    pub selected: PanelSelection,
}

#[derive(Default, Clone, Debug)]
pub struct Output {
    pub stdout: String,
    pub stderr: String,
}

#[derive(Default, Clone, Debug)]
pub struct PanelSelection {
    pub commit: Option<usize>,
    pub file: Option<usize>,
}

pub struct Sut {
    // TODO: pty handle, child handle, frame buffer.
    _placeholder: (),
}

impl std::fmt::Debug for Sut {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.debug_struct("Sut").finish()
    }
}

#[derive(Clone, Copy, Debug)]
pub enum Key {
    Char(char),
    Enter, Esc, Up, Down, PageUp, PageDown,
}

impl World {
    pub fn init_repo(&mut self, _script: &[RepoOp]) -> PathBuf {
        let dir = tempfile::tempdir().expect("tempdir");
        let path = dir.path().to_path_buf();
        self.repo = Some(dir);
        path
    }
}

#[derive(Clone, Debug)]
pub enum RepoOp {
    Branch { name: String, commits: Vec<String> },
    Commit { message: String, files: Vec<(String, String)> },
    Checkout { name: String },
}
```

## `feature-tests/steps/mod.rs`

```rust
// IMPORTANT: proc-macro attributes (`#[given]`, `#[when]`, `#[then]`) do
// NOT propagate through `mod` boundaries in Rust. They must be imported
// in each step sub-module that uses them.

#[path = "repository_discovery_steps.rs"]
pub mod repository_discovery;

// ... one `#[path] mod foo;` line per feature
```

## `feature-tests/steps/<feature>_steps.rs`

```rust
// Skeleton — these steps will fail until the feature is implemented.
// Do not change the .feature file to make this green. Implement the feature.

use cucumber::{given, then, when};

use crate::world::World;

#[given(regex = r#"^<precondition>$"#)]
pub async fn given_<step>(world: &mut World) {
    todo!("<describe what the step should set up>")
}

#[when(regex = r#"^<action>$"#)]
pub async fn when_<step>(world: &mut World) {
    todo!("<describe what the step should drive>")
}

#[then(regex = r#"^<outcome>$"#)]
pub async fn then_<step>(world: &mut World) {
    todo!("<describe what the step should assert>")
}
```

## Run

```sh
cargo test --test cucumber
```

## Notes

- `#[given(expr = "...")]` accepts cucumber expressions; `#[given(regex = r"...")]` accepts raw regex. Mix freely; the spec's step text dictates which to use.
- The `World::run(path)` is the entry point and discovers `*.feature` recursively.
- For TUI tests, the `Sut` skeleton needs `portable-pty` (or `expectrl`); building a real PTY harness is non-trivial. See [tui-hints.md](tui-hints.md) for a working reference implementation, UTF-8/border-stripping heuristics, and stability predicates.
- `harness = false` is **mandatory** for `[[test]]` — without it libtest swallows cucumber's output.
