# Skill: code-review

## Description
Act as a final engineering quality gate for completed changes. Inspects a diff or set of files for correctness, architecture, security, performance, and maintainability issues. Produces a structured report with severity-classified findings. Does not make code changes automatically unless explicitly requested.

## When to use
- After completing a feature, bug fix, or refactor — before reporting done.
- When asked to review a PR, diff, branch, or specific set of files.
- As a quality gate before committing, merging, or deploying.

## When NOT to use
- To fix a bug — use `bug-fix`.
- To implement a feature — use `feature-development`.
- To restructure code — use `refactor`.
- Do not run a full code review on trivial single-line changes.

---

## Default behavior

**Do not make code changes.** Produce a review report. Only apply fixes if the user explicitly requests it.

---

## Required workflow

### Step 1 — Understand scope

Determine what is being reviewed:
1. Is this a diff (branch vs. main), a set of specific files, or a PR?
2. What was the intent of the change? (feature, bug fix, refactor, chore)
3. Are there acceptance criteria or a ticket description?
4. Is there an existing CLAUDE.md that defines project standards?

If reviewing a diff, read the full diff before beginning analysis. Do not stop at the first issue.

### Step 2 — Read the changed code in context

For each changed file:
1. Read the file (not just the diff) to understand surrounding context.
2. Trace how the changed code interacts with its callers and dependencies.
3. Identify what behavior changed vs. what was only restructured.

### Step 3 — Analyze across all dimensions

Work through each dimension below. Not every dimension applies to every change — skip dimensions that are clearly not relevant and note why.

---

## Review dimensions

### Correctness

**Does the implementation satisfy the stated requirement?**
- Does the code do what was asked?
- Are there missing cases or off-by-one errors?
- Are null/undefined values handled at every access point?
- Are array bounds respected?
- Is numeric arithmetic safe (division by zero, integer overflow in relevant contexts)?

**Edge cases**
- What happens when input is empty, zero, negative, or maximum?
- What happens when a list is empty vs. has one item vs. has many?
- What happens when the network is unavailable?
- What happens when the user has no permissions?
- What happens when the user navigates away mid-operation?

**Async safety**
- Are all Promises awaited?
- Are errors from rejected Promises caught and handled?
- Is there a race condition if the user triggers the operation twice?
- Is component state updated after unmount (missing cleanup)?

**Error paths**
- Does every error case surface useful information to the user or calling code?
- Are errors propagated correctly — not swallowed silently?
- Is the error state reset correctly when the operation is retried?

---

### Architecture

**Does the change follow existing architecture?**
- Is new code placed where equivalent existing code lives?
- Is responsibility at the correct layer (UI vs. business logic vs. data layer)?
- Does a new component belong where it was placed, or should it be shared/moved?

**Duplication**
- Does this implement something that already exists elsewhere?
- Should anything in this change be extracted for reuse?

**Abstraction level**
- Is the abstraction too early (one-time use of a "reusable" utility)?
- Is the abstraction absent where clearly needed (copy-pasted logic in three places)?

**Coupling**
- Does the change create unnecessary coupling between unrelated modules?
- Are dependencies going in the correct direction (no circular imports)?

---

### TypeScript / JavaScript quality

**Type safety**
- Is `any` used where a proper type could be defined?
- Are unsafe type assertions (`as SomeType`) justified?
- Are union types handled exhaustively?
- Are optional properties accessed safely (`?.` where needed)?

**Null / undefined handling**
- Are values asserted non-null (`!`) where the guarantee might not hold at runtime?
- Are there unchecked array accesses on data from external sources?

**Async correctness**
- Are `async` functions consistently `await`ed at the call site?
- Are `Promise.all`, `Promise.allSettled`, `Promise.race` used correctly?
- Are floating Promises (fire-and-forget without error handling) intentional?

**Complexity**
- Is any function doing too many things? (Rough signal: hard to describe in one sentence)
- Are there nested callbacks that should be async/await?

---

### React / React Native quality (when applicable)

**Re-renders**
- Does a parent component re-render trigger an unnecessary re-render of a child?
- Are callbacks and objects passed as props created fresh on every render without memoization where it matters?
- Is a component re-fetching data on every render?

**Hooks**
- Are hooks called conditionally or inside loops? (Violation of rules of hooks)
- Are `useEffect` dependency arrays correct and complete?
- Does `useEffect` clean up subscriptions, timers, or listeners on unmount?
- Is `useCallback`/`useMemo` applied where the cost is justified, or applied pointlessly?
- Does `useState` setter receive a function when the new state depends on the previous state?

**List performance**
- Are `FlatList`/`SectionList` used for variable-length lists instead of mapping into a `ScrollView`?
- Are `keyExtractor` and `renderItem` stable (not created inline)?

**Navigation**
- Are new screens registered correctly in the navigator?
- Are navigation params typed correctly?
- Are navigation actions happening in the correct lifecycle moment?

**Platform behavior**
- Does the change behave correctly on both iOS and Android?
- Are platform-specific differences handled?
- Are keyboard avoidance, safe area insets, and back-button behavior considered?

**Accessibility**
- Do interactive elements have `accessibilityLabel`?
- Are `accessibilityRole` and `accessibilityState` set on custom interactive components?

---

### Security

**Secrets and credentials**
- Are API keys, tokens, or passwords hardcoded in source code?
- Are secrets logged?
- Are secrets stored in AsyncStorage/localStorage without encryption (when the project has a secure storage solution)?

**Sensitive data**
- Is personally identifiable or sensitive data logged?
- Is sensitive data passed as URL parameters or route params where it may appear in logs?

**Authentication and authorization**
- Can a user access data or actions they should not be authorized for?
- Is authorization checked server-side (not just hidden in the UI)?

**Input handling**
- Is user-provided input used in a context that could enable injection (SQL, shell, URL, eval)?
- Are URLs constructed from user input validated?

**Storage**
- Are tokens stored with appropriate security level for the platform?
- Are sensitive fields excluded from state persistence where applicable?

**Deep links / URL handling**
- Are deep-link parameters validated before being acted on?
- Can a maliciously crafted deep link trigger unintended navigation or actions?

---

### Performance

**Render performance**
- Are expensive calculations run in the render path without memoization?
- Are large datasets rendered as full lists instead of windowed lists?

**Network efficiency**
- Does the change introduce redundant API calls?
- Is data fetched that is not used?
- Is the change missing request deduplication, caching, or debouncing where clearly appropriate?

**Memory**
- Are subscriptions, listeners, or timers cleaned up on unmount?
- Are large objects retained in state or closure longer than necessary?

**Images and assets**
- Are images loaded at an appropriate resolution (not loading 4K images for thumbnails)?
- Are images cached appropriately?

---

### Testing

**Coverage**
- Do tests exist for the changed code?
- Are the critical code paths (including error paths) tested?
- Are important edge cases covered?

**Quality**
- Do tests assert behavior, not implementation details?
- Are mocks testing the contract, not the internals?
- Will tests fail correctly if the code is broken?
- Are tests brittle (tightly coupled to implementation, easily broken by internal changes)?

**Regressions**
- Does the change touch code that existing tests cover? Do those tests still pass?
- Is there a scenario where the change could silently break something not covered by tests?

---

### Maintainability

**Naming**
- Are names accurate and descriptive?
- Do names follow the project's conventions?
- Are there misleading names (a function named `getUser` that also mutates state)?

**Comments**
- Are comments present where the logic is non-obvious?
- Are comments absent where they just restate the code?
- Are there TODO/FIXME comments that should be resolved or tracked?

**Unnecessary changes**
- Are there changes in the diff that are unrelated to the stated purpose?
- Are there formatting changes mixed with logic changes that obscure the diff?
- Is there dead code introduced (variables assigned but never read, imports never used)?

---

## Severity classification

Classify each finding:

| Severity | Meaning |
|----------|---------|
| **Critical** | Incorrect behavior, data loss, security vulnerability, crash. Must be fixed before merge. |
| **High** | Likely to cause a problem in production; significantly increases maintenance risk. Should be fixed. |
| **Medium** | Probable issue in edge cases or under specific conditions. Recommend fixing. |
| **Low** | Minor quality issue with limited impact. Fix if low effort. |
| **Suggestion** | Improvement worth considering; no obligation to act on it. |

---

## Review report format

```
CODE REVIEW REPORT
==================

Change summary:
[What the change does, in 1-2 sentences]

Intent vs. implementation:
[Does the implementation match the requirement?]

FINDINGS
--------
[SEVERITY] [Category] — [File:line or component]
[Clear description of the issue]
[Why it matters]
[Suggested fix, if straightforward]

RISK AREAS
----------
[Areas of the change that are most likely to cause problems if the review missed something]

TESTING STATUS
--------------
[What is tested, what is not, what is most important to add]

RECOMMENDED ACTIONS
-------------------
Must fix:
- [Critical/High findings]

Should fix:
- [Medium findings]

Consider:
- [Low/Suggestion findings]
```

---

## Common issues in AI-generated or iteratively developed code

These appear frequently and deserve explicit attention:

- Unused imports that were added during iteration and never removed.
- `console.log` statements left from debugging.
- Commented-out code from previous attempts.
- Redundant null checks layered over existing null checks.
- Types widened to `any` to suppress an error rather than fix it.
- Duplicate state (same data stored in two places that can diverge).
- Missing effect cleanup for subscriptions added during iteration.
- Multiple overlapping error-handling layers for the same operation.
- Feature flag or `__DEV__` guard blocks that were never cleaned up.
- Functions introduced during one iteration and then abandoned when the approach changed.

---

## Interaction with other skills

- **codebase-analysis**: Use to understand the project's established standards when reviewing an unfamiliar codebase.
- **feature-analysis**: If an analysis artifact exists, use it as the acceptance criteria reference during the review.
- **feature-development**: Run as the final step after feature implementation.
- **bug-fix**: Run after a fix to verify the change is clean.
- **refactor**: Run after a refactor to confirm behavior was preserved and the diff is clean.
- **ui-ux-review**: Complementary skill — `code-review` covers code correctness; `ui-ux-review` covers visual and UX quality.
- **test-review**: Complementary skill — `code-review` includes a testing dimension at a high level; `test-review` goes deeper on coverage gaps and test quality.
- **project-context**: If the review reveals patterns that should be documented in CLAUDE.md, note them for `project-context`.
