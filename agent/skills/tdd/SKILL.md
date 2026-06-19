---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_ it does it. A good test reads like a specification - "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (like querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" - treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test _imagined_ behavior, not _actual_ behavior
- You end up testing the _shape_ of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes - they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach**: Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

## Workflow

### 1. Planning

When exploring the codebase, use the project's domain glossary so that test names and interface vocabulary match the project's language, and respect ADRs in the area you're touching.

Before writing any code:

- [ ] Confirm with user what interface changes are needed
- [ ] Confirm with user which behaviors to test (prioritize)
- [ ] Identify opportunities for [deep modules](deep-modules.md) (small interface, deep implementation)
- [ ] Design interfaces for [testability](interface-design.md)
- [ ] List the behaviors to test (not implementation steps)
- [ ] Get user approval on the plan
- [ ] **Issue lifecycle**: if the work is tracked in an `issues/` file (or tracker with status field), set its `status` to `in_progress` now, and create/checkout the dedicated branch (`feat/<issue-id>-<slug>` or repo convention)

Ask: "What should the public interface look like? Which behaviors are most important to test?"

**You can't test everything.** Confirm with the user exactly which behaviors matter most. Focus testing effort on critical paths and complex logic, not every possible edge case.

### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

This is your tracer bullet - proves the path works end-to-end.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
COMMIT: Small, focused commit of this green slice
```

Rules:

- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior
- **Commit every green cycle before writing the next test.** Each commit is one RED→GREEN slice: the new (failing-then-passing) test and the minimal implementation that makes it pass. This keeps history bisectable, makes `git bisect` useful, and prevents "horizontal" mega-commits at the end.
- The next RED only starts after the previous commit lands. Never stack uncommitted slices.

Commit message style for a green slice:

```
<area>: <behavior added>

RED→GREEN: <one-line test name> + minimal impl
```

### 4. Refactor

After all tests pass, look for [refactor candidates](refactoring.md):

- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] Apply SOLID principles where natural
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step
- [ ] Commit refactors separately (e.g. `refactor(<area>): <what changed>`), never mixed with a green-slice commit

**Never refactor while RED.** Get to GREEN first.

### 5. Close the Issue

Only do this when every cycle is GREEN, refactor is done, and each cycle is already committed:

- [ ] Run the full verification suite (build, vet, lint, full test run) one more time
- [ ] Tick every unchecked box in the issue's `Acceptance Criteria`
- [ ] Update the issue's `status` field to `done` (or the value the tracker expects)
- [ ] Make a final commit with the issue-status/docs change (e.g. `docs(ISSUE-001): mark done`) — no source code in this commit

If any acceptance criterion cannot be ticked, the issue stays `in_progress` and a follow-up note is added inside the issue explaining the gap.

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
[ ] Test was RED, then went GREEN
[ ] Green slice committed (one RED→GREEN = one commit) before next RED
```

## Checklist When Closing the Issue

```
[ ] Issue status moved to in_progress before the first RED
[ ] Dedicated branch created/checked out
[ ] Full verification (build + vet + tests) green
[ ] Every Acceptance-Criteria box ticked
[ ] Issue status moved to done
[ ] User has approved the diff before any commit
```
