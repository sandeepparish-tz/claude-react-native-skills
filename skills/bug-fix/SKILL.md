# Skill: bug-fix

## Description
Systematically diagnose and fix defects through evidence-based root-cause analysis. Avoids speculative changes, workarounds, and unrelated modifications.

## When to use
- When a specific defect, crash, regression, or unexpected behavior is reported.
- When something that previously worked has stopped working.
- When the application behaves differently from what the requirement intended.

## When NOT to use
- Implementing new functionality — use `feature-development`.
- Improving code structure without fixing a defect — use `refactor`.
- Reviewing a completed change — use `code-review`.

---

## Required workflow

### Step 1 — Understand the bug

Collect all available information before reading any code:
- What is the observed behavior?
- What is the expected behavior?
- How is it reproduced? (exact steps, minimal case if known)
- When did it start occurring? (always present, or introduced by a recent change?)
- Which environments is it present in? (dev, staging, production, specific OS, specific device, specific platform)
- Is there a stack trace, error log, or crash report? Read it in full.
- Is there a recent commit or PR that correlates with the onset?

If a stack trace is provided, read it top-to-bottom before doing anything else. Identify the exact file and line the error originates from, not just where it was caught.

### Step 2 — Reproduce (or confirm reproducibility)

Before fixing anything:
- Can the bug be reproduced with the information provided?
- If yes, identify the minimal code path that triggers it.
- If no, state explicitly what information is missing and ask for it.

Do not proceed to fix a bug you cannot confirm understanding of.

### Step 3 — Collect evidence

Trace from the symptom backward to the cause:

1. Start at the reported failure point (error message, wrong UI state, wrong data).
2. Read the relevant code: the component, function, hook, or service directly involved.
3. Trace data flow backward: where does this data come from? What transforms it?
4. Trace execution flow: what triggers this code path?
5. Look for recent modifications: `git log --follow -p <file>` on involved files if git is available.

Evidence types to gather:
- The exact error message and stack trace.
- The data state at the point of failure (what value was unexpected).
- The expected data state.
- The code responsible for producing the unexpected state.
- Any conditions that make the bug appear or disappear (device, env, timing, network).

When you have read enough to identify one clear, evidenced candidate root cause, stop gathering and move to Step 4. Do not continue reading unrelated code.

### Step 4 — Identify root cause

State the root cause explicitly before writing any fix:

```
ROOT CAUSE
----------
What: [The specific incorrect behavior in the code]
Where: [File and line/function]
Why: [The logic or assumption that is wrong]
Evidence: [What you observed that confirms this]

SYMPTOMS (vs root cause)
------------------------
[List the observable symptoms — these are consequences, not causes]
```

Distinguish:
- **Root cause**: The actual incorrect code or logic.
- **Symptom**: What the user sees or what the error message says.
- **Trigger**: The condition that exposes the root cause (may not always be present).

If you cannot identify a single root cause with evidence, state what is unknown and what additional information would resolve it. Do not guess.

### Step 5 — Determine the fix

Before writing code, define the fix:

```
PROPOSED FIX
------------
Approach: [What the fix changes and why that addresses the root cause]
Files affected: [List]
Risk: [Could this fix affect anything else?]
Alternative considered: [If another approach exists, briefly explain why this one is better]
```

Rules for choosing a fix:
- Fix the root cause, not the symptom.
- Make the smallest change that correctly resolves the root cause.
- Do not refactor surrounding code as part of the fix.
- Do not change behavior beyond the scope of the bug.
- If the fix requires understanding a subsystem you have not analyzed, analyze it before proceeding.

### Step 6 — Implement the fix

Apply only the changes described in the proposed fix.

During implementation:
- Do not rename variables, reorganize code, or clean up formatting outside the fix.
- Do not add error handling for unrelated scenarios.
- Do not add logging beyond what is needed to diagnose this specific bug.
- Do not add timeouts, retries, or fallbacks as workarounds for a root cause you have not identified.
- Do not disable validation, type checks, or guards.
- Do not swallow errors.

### Step 7 — Regression check

After the fix:
1. Mentally trace the fixed code path with the reproduction case — does it now behave correctly?
2. Identify any other code path that uses the same function/component/data — does the fix affect those?
3. Run the test suite if available.
4. Run type checking and linting if available.
5. If tests exist for the fixed area, verify they pass. If no test exists, consider whether one should be added (only if the project has tests for similar scenarios).

### Step 8 — Review the diff

Before reporting done:
1. Re-read every changed line.
2. Confirm no unrelated changes were made.
3. Confirm no debug code was left in.
4. Confirm the fix is complete — no half-applied changes.

### Step 9 — Report

Summarize:
- Root cause identified.
- Fix applied.
- Files changed.
- How to verify the fix.
- Any related issues noticed (do not fix them inline; flag them separately).

---

## Bug categories and investigation approach

### UI / rendering bug
Start at the component. Read it and its props. Trace what data/state drives the incorrect rendering. Look for: incorrect conditional rendering, missing state update, stale closure, incorrect key on a list item.

### State bug
Trace backward from the incorrect state value: who sets it, what triggers the setter, what the expected trigger is. Look for: incorrect initial state, missing reset on unmount/navigation, state mutation instead of replacement, incorrect dependency array in an effect.

### Navigation bug
Read the relevant route/screen registration. Trace the navigation call. Look for: missing screen registration, incorrect params passing, screen rendered before data is available, navigator not reset correctly.

### Async / timing bug
Trace the async operation: initiation, success path, error path, cancellation path. Look for: missing loading state, unhandled rejection, race condition between two async operations, effect that does not handle cleanup, stale closure over async data.

### API bug
Check request: URL, method, headers, body construction. Check response: status handling, error parsing, data mapping. Look for: incorrect base URL, missing auth header, incorrect JSON field name, API returning 4xx/5xx that is not being surfaced.

### Caching bug
Identify whether the project caches responses. If yes, look for: stale cache not being invalidated, incorrect cache key, optimistic update not being rolled back on failure.

### Race condition
Look for: two operations that modify the same state and can complete in either order, event listeners that are attached multiple times, effects that depend on rapidly changing values.

### Platform-specific bug (Android / iOS)
Confirm on which platform the bug occurs. Check: platform-specific file variants (`.android.ts`, `.ios.ts`), platform conditionals (`Platform.OS`), native module availability, OS version differences, specific hardware behavior (notch, back button, keyboard behavior, permissions model).

### Performance regression
Measure before diagnosing. Look for: unnecessary re-renders (use React DevTools if available), expensive calculations in render path, large list without windowing, image/asset size.

### Lifecycle / mount-unmount bug
Read the component's effects. Look for: effect that runs when component is unmounted, missing cleanup in useEffect return, state update after unmount.

### Authentication / session bug
Trace the auth state and how it is checked. Look for: token not being refreshed, stale token in cache, incorrect redirect after login/logout, session state not cleared on logout.

### Build / configuration bug
Read the build config files relevant to the failure. Do not guess at config values — read the actual config. Look for: environment variable not set, incorrect bundle identifier, signing issue, missing Podfile dependency.

---

## Explicitly prohibited actions

Do not do any of the following as part of a bug fix:

- Change code that is unrelated to the root cause.
- Refactor working code.
- Add arbitrary `setTimeout` or `delay` to fix timing issues (this masks the root cause).
- Add `try/catch` that swallows the error without surfacing it.
- Disable TypeScript type checking (`// @ts-ignore`, `as any`) to suppress an error symptom.
- Add excessive defensive checks around code that should be valid (treat the symptom).
- Add a new dependency to solve a problem that the existing codebase already handles elsewhere.
- Change multiple systems simultaneously without a clear, evidenced reason each one contributes to the bug.
- Mark a bug as fixed before confirming the root cause.

---

## When evidence is insufficient

State explicitly:
```
INSUFFICIENT EVIDENCE
---------------------
What is known: [...]
What is unknown: [...]
What is needed to determine root cause: [...]
```

Then ask for the missing information or identify a targeted investigation step that would produce it. Do not implement a speculative fix.

---

## Interaction with other skills

- **codebase-analysis**: Run a scoped analysis first if the bug is in an unfamiliar subsystem.
- **feature-development**: Do not implement new features as part of a bug fix. If the fix reveals a missing feature, flag it separately.
- **refactor**: Do not refactor as part of a fix. If the bug-prone code needs structural improvement, note it and address separately.
- **test-review**: After fixing, run `test-review` to confirm a regression test was added or that existing tests cover the fixed scenario.
- **code-review**: Run after fix to ensure the change is clean and does not introduce new issues.
