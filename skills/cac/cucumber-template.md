# Cucumber Test Skeleton Templates

Generate **red** Cucumber tests from CACs in a PRD. The implementation does
not exist yet. The contract: the CAC text in the PRD is the stable spec;
the test skeleton turns green when the feature ships — without any change
to the CAC.

## Choose target framework first

Before generating, ask the user which language/framework the project uses:

| Language | Framework | Template |
|---|---|---|
| Rust | `cucumber` crate | [cucumber-rust.md](cucumber-rust.md) |
| Java | `cucumber-jvm` | [cucumber-java.md](cucumber-java.md) |
| Python | `pytest-bdd` or `behave` | [cucumber-python.md](cucumber-python.md) |
| Go | `godog` | [cucumber-go.md](cucumber-go.md) |
| Other | — | ask the user for the exact package + runner, then assemble from the nearest neighbour |

After the user picks, copy the corresponding template into the project,
then adapt the Gherkin, World, and step bodies to the specific CACs from
the PRD. Do **not** copy the Rust template into a Java project.

## Common contract (all frameworks)

1. **Layout** (canonical — adapt only if the project dictates otherwise):
   ```
   <project>/
   ├── feature-tests/
   │   ├── README.md                  # "do not change the CAC to make tests green"
   │   ├── world.<ext>
   │   ├── <feature-slug>.feature     # 1:1 from the PRD
   │   └── steps/...
   └── <runner>                        # tests/cucumber.rs | *Test.java | ...
   ```
2. **Step-definition contract.** First line of every step file: `// Skeleton — these steps will fail until the feature is implemented. Do not change the .feature file to make this green. Implement the feature.`
3. **World / test context.** Holds the test repo, SUT handle, captured stdout/stderr, last frame, and any per-panel state. Steps mutate it; assertions read from it.
4. **SUT placeholder.** For TUI tests, the SUT skeleton is a struct with a TODO. The user implements spawn, send_key, read_frame, terminate when the harness is built.
5. **Run command.** Every framework's `Run` section in its sub-template shows the exact command. The skill must verify the test compiles and is discovered — failure to compile is a skill bug, not a CAC bug.

## Framework-specific templates

- [Rust (`cucumber` crate)](cucumber-rust.md)
- [Java (`cucumber-jvm`)](cucumber-java.md)
- [Python (`pytest-bdd` / `behave`)](cucumber-python.md)
- [Go (`godog`)](cucumber-go.md)

## Special cases

- [TUI SUTs](tui-hints.md) — if the System Under Test is a Terminal User
  Interface, the step harness must drive a real PTY. This is non-trivial;
  see the reference for a working harness, UTF-8/border-stripping
  heuristics, and stability predicates.
