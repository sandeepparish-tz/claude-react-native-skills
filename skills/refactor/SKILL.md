# Skill: refactor

## Description
Safely improve existing code structure, clarity, and maintainability without changing observable behavior. Particularly effective at cleaning up code that has accumulated debt through AI-generated patches, iterative development, or rushed delivery.

## When to use
- When code has grown complex, duplicated, or hard to understand and a specific improvement is needed.
- When removing dead code, unused imports, or obsolete logic.
- When consolidating duplicate implementations of the same concept.
- When explicitly asked to clean up, consolidate, or simplify.

## When NOT to use
- To fix a defect — use `bug-fix`.
- To add new functionality — use `feature-development`.
- To review a change — use `code-review`.
- As a side task during a bug fix or feature development. Note the refactor opportunity and address it separately.

---

## Core principle

> Preserve behavior unless the task explicitly includes behavior changes.

A refactor that changes behavior is a bug. Verify behavior is preserved before and after every change.

---

## Required workflow

### Step 1 — Understand current behavior

Before changing any code:
1. Read the code to be refactored completely.
2. Understand what it does — not what it looks like, what it does.
3. Identify all callers/consumers of the code being changed.
4. Identify all side effects: state changes, API calls, navigation actions, events emitted.
5. Note what tests exist for this code and what they cover.

Document current behavior explicitly before proceeding:
```
CURRENT BEHAVIOR
----------------
[What the code does, in concrete terms]

CALLERS / CONSUMERS
-------------------
[Every place that uses this code]

EXISTING TESTS
--------------
[Test files and what they cover, or "none"]
```

### Step 2 — Identify the target architecture

Identify what the code should look like after refactoring:
1. What is the correct responsibility for this module/component/function?
2. What is the project's established pattern for this type of code?
3. Are there well-structured examples elsewhere in the codebase to align with?

Do not invent a new architecture. Align with what already exists and works in the project.

### Step 3 — Classify what needs to change

Sort every identified issue into one of three categories:

**Necessary** — Directly causes maintainability problems, confusion, or correctness risk:
- Duplicate logic that will diverge if one copy is updated without the other.
- Dead code that is demonstrably never executed.
- Unused imports, variables, or files.
- Code that is structurally broken but happens to work (e.g., a race condition that rarely fires).
- Incorrect separation of responsibilities causing meaningful coupling issues.

**Useful** — Clear improvement but the code works adequately without it:
- Overly complex conditionals that can be simplified.
- Inconsistent naming that causes reader confusion.
- Missing extraction of a repeated code block used 3+ times.
- An abstraction that has been outgrown and should be flattened.

**Optional / stylistic** — Preference, not improvement:
- Formatting that tools already handle.
- Naming that is slightly different from what you would choose.
- Structure that differs from your preferred pattern but matches the project's existing pattern.

**Only implement Necessary changes by default.** Include Useful improvements only when they are clearly within scope. Never implement Optional/stylistic changes.

### Step 4 — Plan the refactor

Write the plan before writing any code:

```
REFACTOR PLAN
-------------
Target: [What is being refactored]

Changes (Necessary):
- [Specific change] — [Reason]

Changes (Useful, if in scope):
- [Specific change] — [Reason]

NOT changing (and why):
- [Tempting change excluded] — [Reason: out of scope, stylistic, risk]

Behavior preserved:
- [How you will verify nothing changed]

Files affected:
- [List]
```

### Step 5 — Implement incrementally

Make changes in small, logically isolated steps:

1. One type of change at a time (e.g., remove dead code, then consolidate duplicates, then simplify logic).
2. After each logical unit of change, verify: does the code still compile? Do types still check out?
3. Do not mix refactor changes with behavior changes in the same edit.

During implementation:
- Remove dead code completely. Do not comment it out.
- Remove unused imports. Do not leave them "in case they are needed later."
- Do not add new abstractions unless the refactor explicitly calls for consolidation.
- Do not rename things just because you prefer a different name.
- Do not reformat code that linting/formatting tools will handle automatically.

### Step 6 — Verify behavior preservation

After implementation:
1. Re-read the refactored code end-to-end.
2. Trace the same execution paths you identified in Step 1.
3. Confirm each caller/consumer still works correctly.
4. Run the test suite if available.
5. Run type checking and linting.
6. If existing tests fail, do not suppress them — investigate why. A failing test after a refactor means either the behavior changed (a bug) or the test needs updating because it was testing implementation details (assess carefully before changing the test).

### Step 7 — Review the diff

Before reporting done:
1. Read every changed line.
2. Confirm no behavior was accidentally changed.
3. Confirm the diff contains only what was planned.
4. Confirm no debug code was introduced.
5. Confirm no unnecessary new files were created.
6. Remove any change that is not clearly an improvement.

---

## Common refactor targets in iteratively developed codebases

### Duplicate logic
Search for: repeated patterns performing the same operation (data transformation, API calls, state updates) across multiple files. Consolidate into a single source of truth. Verify every usage is updated.

### Dead code
Look for: functions never called, components never rendered, imports never used, branches of a conditional that can never be true, feature flags that were never removed, config values that are never read. Remove completely.

### Unnecessary state
Look for: state that can be derived from other state or props (`useMemo` candidate), state that is only ever set to a constant, state that is reset immediately after being set, state that duplicates server data locally for no purpose.

### Unnecessary effects
Look for: `useEffect` that runs on every render and does not need to, `useEffect` that reacts to a value but only ever does the same thing regardless of that value, effects that could be replaced with event handlers, effects that set state based on other state (often causes render loops).

### Unnecessary memoization
Look for: `useMemo`/`useCallback`/`React.memo` applied to trivially cheap operations or to code that changes every render anyway (making the memoization pointless). Remove when the cost of memoization exceeds its benefit.

### Excessive abstraction
Look for: a utility function used exactly once with no prospect of reuse, a HOC wrapping a single component, a context providing a single value that could be a prop, an abstraction that makes the call site harder to understand than the underlying code. Flatten when appropriate.

### Nested conditionals
Refactor deeply nested `if/else` trees using early returns or guard clauses, matching the style used in the rest of the project.

### Inconsistent patterns
When the same operation is done two different ways in adjacent code with no reason for the difference, consolidate to the project's established pattern.

### AI-generated patch debt
Common patterns introduced by iterative AI-generated changes:
- Multiple layers of try/catch around the same operation.
- Duplicate API calls fetching the same data in the same component.
- Redundant null checks for values that are guaranteed non-null at that point.
- Types broadened to `any` or `unknown` to suppress an error rather than fix it.
- Comments that describe what the code does rather than why (remove unless the why is non-obvious).
- Fallback values on top of fallback values for the same field.
- Imports from paths that no longer exist (kept working by a coincidental re-export).
- Feature flags or `__DEV__` guards that are never removed after the experiment ended.

---

## What refactoring is NOT

- **Not rewriting**: A refactor preserves behavior and improves structure. A rewrite changes both. If the required improvement cannot be made without significant behavioral risk, discuss before proceeding.
- **Not a style exercise**: If the code works and is understandable, do not change its style.
- **Not an opportunity to introduce new libraries**: If a refactor reveals that an existing library is insufficient, that is a separate architectural decision.
- **Not cleanup for its own sake**: Every change must make the code observably clearer, safer, or less duplicated. If you cannot articulate the concrete improvement, do not make the change.

---

## Interaction with other skills

- **codebase-analysis**: Run before a large-scale refactor to map what is safe to change.
- **feature-development**: If a refactor reveals missing functionality, flag it separately; do not implement it inline.
- **bug-fix**: If a refactor uncovers a defect, flag it separately; do not fix it as part of the refactor.
- **test-review**: Run after refactor to confirm existing tests still pass and coverage is adequate for the changed code.
- **code-review**: Run after refactor to catch unintended behavior changes and verify cleanliness.
