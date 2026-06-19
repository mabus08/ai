---
name: prd-to-issues
description: Breaks down a PRD into agent-optimised markdown issues stored in the project's /issues directory. Use when user wants to convert a PRD into a task list, a set of implementation issues, or "break down the PRD".
---

# PRD to Issues

Break down a PRD into independently-grabbable, agent-optimised markdown issues using vertical slices (tracer bullets).

## Process

### 1. Gather Context
Read the provided PRD. Identify the core user stories and their associated acceptance criteria (ACs).

### 2. Explore Codebase (Optional)
Explore the codebase to ensure issue titles and descriptions use the project's domain glossary and respect existing architectural decisions (ADRs).

### 3. Draft Iteration-Sized Slices
Break the PRD into issues following the **Ralph iteration model**: each issue must be completable in one context window.

- **Preference**: Use **vertical slices (tracer bullets)** that cut through all layers (DB $\rightarrow$ API $\rightarrow$ UI) to deliver a complete, demoable feature path.
- **Hard Limit**: If a vertical slice is too large to be described in 2-3 sentences, it MUST be split into smaller issues (even if this means falling back to horizontal slices, e.g., separate DB and UI issues).
- Each slice must be independently verifiable.
- Prefer many thin slices over few thick ones.

### 4. Quiz the User
Present the proposed breakdown as a numbered list. For each slice, show:
- **Title**: Short descriptive name.
- **Dependencies**: Which other slices must complete first.
- **User Stories**: Which specific PRD user stories this addresses.
- **Type**: Vertical (End-to-End) or Horizontal (Layer-specific).

Ask the user:
- Does the granularity feel right? (Is any story too big for one iteration?)
- Are the dependency relationships correct?
- Should any slices be merged or split further?

Iterate until approved.

### 5. Publish Issues to `/issues/`
Create the `/issues` directory if it does not exist. For each approved slice, create a markdown file:

**Filename**: `/issues/[ID].md` (e.g., `/issues/ISSUE-001.md`)

**Content Template**:
```md
---
id: ISSUE-XXX
name: "Short Descriptive Title"
status: todo
dependencies: [ISSUE-YYY, ISSUE-ZZZ]
estimated_effort: 
---

## PRD Reference
Reference to the specific section or user story in the PRD.

## Description
A concise description (2-3 sentences) of the change. Describe the end-to-end behavior.

## Acceptance Criteria
- [ ] AC 1 from PRD (must be verifiable/concrete)
- [ ] AC 2 from PRD
- [ ] Typecheck passes
- [ ] Verify in browser using dev-browser skill (required for UI changes)
- [ ] Tests pass (required for logic changes)
```

**Note**: 
- Set `status` to `todo` by default.
- Ensure dependencies are correctly mapped to the generated IDs.
- Create an `index.md` in the `/issues/` folder listing all created issues and their status if not already present.
