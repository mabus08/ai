---
name: cac
description: Define Customer Acceptance Criteria (CAC) for a PRD feature in Gherkin, or generate executable Cucumber test skeletons from them. Use when user mentions "Customer Acceptance Criteria", "CAC", "Abnahmekriterien", "Kundenabnahme", "Gherkin", "Cucumber", "Feature-Tests", or wants E2E-style acceptance tests written from a PRD.
---

# Customer Acceptance Criteria (CAC)

## What CAC is

A **CAC** is one Gherkin scenario that captures **one complete customer use case** for a feature. CACs live in the PRD alongside User Stories and their Acceptance Criteria (ACs), but live **one or two steps higher on the test pyramid**:

| Level | Lives in | Granularity | Style |
|---|---|---|---|
| AC | PRD, per User Story | per behavior (unit/integration) | free-form or per-story |
| **CAC** | **PRD, per Feature** | **per use case (E2E/acceptance)** | **Gherkin, cucumber-executable** |
| E2E test | `tests/`, per Feature | per use case | executable |

**Rule of thumb**: an AC verifies that a *part* works; a CAC verifies that the *customer use case* is satisfied.

## When to use

1. **Defining CACs** — user asks to add CACs to a new or existing PRD.
2. **Generating Cucumber tests** — user asks to derive test skeletons from CACs in a PRD.

## Granularity rules (binding)

A good CAC scenario:

- **One scenario = one customer use case** (start-to-finish). If a step changes the focused panel and a follow-up step would be useless without it, keep them together.
- **No implementation details.** Forbid: "rendered with ratatui TestBackend", "uses `git show-branch`", "calls `app::state` reducer", "the internal `Worker` thread". Use observable, customer-facing language.
- **No premature numeric/structural precision.** Forbid: "25/75 horizontal split", "first 7 characters of SHA", "two line-number columns". Replace with customer-perceivable: "the diff panel is wider", "line numbers are visible so specific lines can be referenced", "each entry shows hash and subject".
- **Cucumber-executable.** Each `Given/When/Then` maps to a step definition. Steps are imperative, present tense.
- **Ubiquitous language.** Use the project's glossary terms (e.g. `Active Panel`, `Base Detection`, `Branch Commits`, `Changed Files`, `Status`) consistently.
- **Edge cases stay out** (initial commit, renames, etc.) — those are ACs, not CACs.

## Workflow

### Mode A — Define CACs for a PRD

1. Read the PRD. Identify features (groups of related User Stories).
2. For each feature, draft CAC scenarios applying the granularity rules above.
3. Each scenario covers **one complete use case** end-to-end.
4. Place CACs in a new `## Customer Acceptance Criteria` section in the PRD, grouped by feature with a `Feature: <name>` Gherkin header.
5. Present the draft to the user. Iterate. See [examples.md](examples.md) for the gitreview reference set.

**PRD template**: see [prd-template.md](prd-template.md).

### Mode B — Generate Cucumber test skeletons from CACs

1. Locate the PRD and the CAC section. Parse all `Feature:` blocks and their scenarios.
2. **Ask the user** for project-specific decisions:
   - **Cucumber framework** (default: `cucumber` crate for Rust; or `cucumber-js`, `cucumber-jvm`, `behave`, `pytest-bdd`, etc.)
   - **Execution mode** (default: `cargo test --test <name>` for Rust; project-specific otherwise)
   - **Feature file language** (default: English; match the PRD)
3. For each feature, generate the framework's standard files under `feature-tests/`:
   - `feature-tests/<feature-slug>.feature` — copy of the Gherkin from the PRD.
   - `feature-tests/world.<ext>` — shared test context, with a `Sut` placeholder for TUI/CLI tools and a `World` struct holding the test repo, captured output, last rendered frame, panel state.
   - `feature-tests/steps/<feature-slug>_steps.<ext>` — step-definition skeleton with `TODO`/`pending` markers in the body.
4. Steps are **not required to pass** — implementation hasn't started. They must be **structured so that, once the feature is implemented, the test turns green without changing the CAC** in the PRD. This is the contract: CACs are the stable spec, tests are the executable reflection.
5. Wire up the framework's runner entry point.
6. Run the test command once to confirm everything compiles and the features are discovered. **Expect red** — every step body is unimplemented. Red-but-compiling is the success criterion.
7. Do **not** rewrite the test bodies to make them pass. The next engineer picks this up during implementation.

**Framework templates**: see [cucumber-template.md](cucumber-template.md) for the index, then the specific sub-template (e.g. [cucumber-rust.md](cucumber-rust.md)).

**TUI SUTs**: if the System Under Test is a Terminal User Interface, the
harness must drive a real PTY. This is non-trivial; see
[tui-hints.md](tui-hints.md) for a working reference, UTF-8/border
heuristics, and stability predicates.

## Definition of done

- [ ] CAC section added to PRD, grouped by `Feature:` headers.
- [ ] Each scenario: one use case, no implementation detail, ubiquitous-language terms.
- [ ] User has reviewed and approved the CACs.
- [ ] If Mode B: project-specific questions answered, test files generated under `feature-tests/`, runner wired, tests compile and are discovered (red is fine).
- [ ] If Mode B: a one-liner comment in the generated step files states "Steps will fail until feature is implemented. Do not change the CACs in the PRD to make tests green — change the implementation."

## Anti-patterns

- **Splitting one use case into many scenarios.** "Press j", "press k", "press g", "press G" as four scenarios is testing internals. Combine into "navigate a long list line-by-line and jump to top/bottom".
- **Restating the AC in Gherkin syntax.** Gherkin is not a formatting rule; the test pyramid is the point.
- **Acronyms for things not yet built.** Don't name internal types or modules in a CAC.
- **Over-precise layout.** "Exactly 25 columns" is a constraint test, not a customer test. "The diff panel has the most space" is.
- **Mixing ACs and CACs.** ACs stay with their User Stories. CACs get their own section.
