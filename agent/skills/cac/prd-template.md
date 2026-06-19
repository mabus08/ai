# PRD Template — Customer Acceptance Criteria Section

Insert this section into the PRD, **after** `## User Stories` and **before** `## Implementation Decisions`.

```markdown
## Customer Acceptance Criteria

Customer Acceptance Criteria (CAC) are end-to-end acceptance tests written in
Gherkin. They cover one complete customer use case per scenario and live one
level above Acceptance Criteria (which verify individual behaviors of a User
Story). CACs are the stable, customer-facing spec; Cucumber test skeletons in
`feature-tests/` are their executable reflection and may change, but the CAC
section in this PRD must not change to make tests green.

### Feature: <feature name in ubiquitous language>

As a <role>
I want <capability>
So that <value>

**Scenario: <use-case name, declarative>**
```gherkin
Given <preconditions, present state>
And   <additional preconditions>
When  <actor performs the action>
And   <follow-up action if the use case requires it>
Then  <observable, customer-facing outcome>
And   <additional outcome>
```

**Scenario: <next use case>**
...

### Feature: <next feature>

...
```

## How to phrase scenarios

- **One scenario = one use case.** A use case is a complete customer journey.
  If steps depend on each other, they belong together.
- **No implementation words.** Forbidden in CAC: `ratatui`, `TestBackend`,
  module names, struct names, internal thread names, function names,
  `git show-branch`, `merge-base`, command-line flag names unless they are
  part of the customer-facing surface.
- **No over-precise numbers** that test the constraint instead of the
  experience. Replace "25/75 split" with "the diff panel has the most space".
- **Use the ubiquitous language** from the project's glossary. In this
  project: Branch Commits, Base Branch, Base Detection, Changed Files, Status,
  Active Panel, Commits/Files/Diff Panel.
- **Out-of-scope edge cases stay out.** Renames, initial commits, large
  histories, etc. are AC material, not CAC material.

## Worked example (excerpt from a TUI git review tool)

```gherkin
### Feature: Browsing commits

As a developer
I want to see and select commits in the Commits Panel
So that I can drive the review flow

**Scenario: Commits are listed in reverse chronological order with short hash and subject**
Given the current branch has commits A, B, C in that order (A oldest)
When  gitreview starts
Then  the Commits Panel shows C, B, A from top to bottom
And   each entry is displayed as "<short-hash> <subject>"
And   no author, date, or body is shown

**Scenario: Navigate commits with j/k and select one with Enter**
Given focus is in the Commits Panel
And   the panel lists 3 commits
When  the user presses "j" then "k"
Then  the selection moves down then up
When  the user presses "Enter"
Then  that commit becomes the selected commit
And   the Files Panel shows the changed files of that commit
And   focus moves to the Files Panel
```

Notice:

- Both scenarios are about the *same* panel, but they are **different use
  cases** from the customer's perspective: "what do I see when I open the
  tool" vs. "how do I drive a review". Not split into "press j", "press k",
  "press Enter".
- No mention of `app::state`, `Action::SelectCommit`, `ratatui` or
  `TestBackend`.
- The "first 7 characters" detail is omitted because the customer perceives
  it as "a short hash" — the actual length is a visual convention, not a
  customer concern.
