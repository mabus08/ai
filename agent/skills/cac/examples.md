# Examples — CACs for a TUI git review tool

Reference set derived from the gitreview PRD. Use as a calibration sample
when drafting CACs for a similar tool. Do not copy verbatim into a different
project — every project has its own ubiquitous language and customer use
cases.

## Example 1: Picking the wrong granularity

### Too granular (3 scenarios for one use case) — DO NOT

```gherkin
Scenario: Pressing j moves selection down
Scenario: Pressing k moves selection up
Scenario: Pressing Enter selects the commit and moves focus to the Files Panel
```

Why wrong: from the customer's perspective, "navigate the commit list and
pick one" is **one** use case, not three. The keyboard bindings are an
implementation detail of that one use case.

### Right granularity (1 scenario for one use case) — DO

```gherkin
Scenario: Navigate commits with j/k and select one with Enter
Given focus is in the Commits Panel
And   the panel lists 3 commits
When  the user presses "j" then "k"
Then  the selection moves down then up
When  the user presses "Enter"
Then  that commit becomes the selected commit
And   the Files Panel shows the changed files of that commit
And   focus moves to the Files Panel
```

## Example 2: Phrasing — implementation vs customer

### Implementation detail — DO NOT

```gherkin
Then  the Commits Panel uses the app::state::reducer
And   the action Action::SelectCommit(2) is dispatched
And   the diff::parser tags lines as Added/Removed/Context
```

### Customer-facing — DO

```gherkin
Then  that commit becomes the selected commit
And   the Files Panel shows the changed files of that commit
And   the diff shows added lines in one color and removed lines in another
```

## Example 3: Over-precise numbers vs perceived experience

### Over-precise — DO NOT

```gherkin
Then  the layout uses a 25/75 horizontal split
And   the left column uses a 60/40 vertical split
And   the short hash is the first 7 characters of the SHA
```

### Customer-perceivable — DO

```gherkin
Then  the Diff Panel is wider than either of the two left panels
And   each entry in the Commits Panel is displayed as "<short-hash> <subject>"
```

## Full example set (9 features, 17 scenarios)

The full set covers: repository discovery, base branch detection, browsing
commits, inspecting changed files, reading a diff, in-panel navigation,
panel focus, search, help, quit, and responsiveness. The complete file is
in the project PRD `docs/prd/0001-gitreview-tui-mvp.md`. When using it as
reference, watch for:

- **Feature headers** group related use cases.
- **One scenario = one complete use case** even when the use case has
  several actions in sequence.
- **No type/struct/function names** appear in any step.
- **No "exact pixel / character / column" precision** that the customer
  would phrase as "looks right".
