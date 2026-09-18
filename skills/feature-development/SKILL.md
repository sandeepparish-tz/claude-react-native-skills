# Skill: feature-development

## Description
Implement new functionality in an existing project using disciplined, evidence-first workflow. Reuses existing architecture, patterns, and utilities. Avoids inventing new approaches when existing ones apply.

## When to use
- When asked to add a new feature, screen, component, API integration, or user-facing behavior.
- When asked to extend existing functionality in a non-trivial way.
- After a `feature-analysis` artifact has been produced and reviewed — consume the artifact rather than re-deriving the same conclusions.

## When NOT to use
- Fixing a defect in existing behavior — use `bug-fix`.
- Improving existing code structure without changing behavior — use `refactor`.
- Reviewing a completed implementation — use `code-review`.
- When the scope is purely a content or copy change to an existing file.

---

## Required workflow

### Step 1 — Understand the requirement

Before touching any file:
1. Restate the requirement in your own words.
2. Identify what the feature does, who it is for, and what done looks like.
3. Identify ambiguities or contradictions. List them explicitly.
4. Determine whether any ambiguity would materially change the implementation. If yes, ask focused questions before proceeding. Do not ask about things you can reasonably infer from context.

Questions to answer before planning:
- Is this a new screen, a new component, a new API call, or a combination?
- Is there any existing feature that is similar or overlapping?
- Are there specific edge cases, error states, or permissions involved?
- Are there platform-specific requirements (iOS vs. Android)?
- Is testing explicitly required or expected?

### Step 2 — Check for a feature-analysis artifact

If a `feature-analysis` artifact was produced for this feature:
1. Read it completely before doing anything else.
2. Use its implementation plan as the starting point — do not re-derive what it already established.
3. Use its "Reusable Existing Code" and "Required Changes" sections to inform Steps 3 and 4 below.
4. Note any open questions from the artifact and resolve them before implementing.

If no artifact exists, proceed with the investigation below.

### Step 2b — Analyze the relevant codebase area (when no artifact exists)

Do not start a full `codebase-analysis` unless you are entering a completely unfamiliar project. Instead, scope the investigation:

1. Find the closest existing feature to the one being built. Read it end-to-end.
2. Identify the components, hooks, services, and utilities it uses.
3. Understand how data flows: source → state → UI.
4. Understand how navigation works for similar screens/flows.
5. Understand how the existing feature handles: loading, error, empty, success states.
6. Identify the testing approach for similar features.

Search for existing implementations before planning anything new:
```
grep -r "relevant keyword" src/ --include="*.ts" --include="*.tsx" -l
```

### Step 3 — Identify what to reuse

List explicitly:
- **Reuse**: Existing components, hooks, utilities, services, types.
- **Extend**: Existing APIs, stores, navigation structures that need a new entry.
- **Create new**: Only what cannot be reasonably achieved by reuse or extension.

If something similar already exists but has a different name or location, investigate before creating a new version. Duplicate implementations are a common failure mode.

### Step 4 — Create an implementation plan

Write a concise plan before writing any code:

```
IMPLEMENTATION PLAN

Requirement summary:
[1-2 sentences]

Files to create:
- [path] — [purpose]

Files to modify:
- [path] — [what changes]

Reused:
- [component/hook/utility] from [path]

New dependencies required:
- [none | package — reason]

Edge cases to handle:
- [list]

Questions / assumptions:
- [anything that cannot be resolved from the codebase]
```

Review this plan before implementing. If the plan requires a new dependency, an architectural change, or a significant new abstraction, explicitly justify why the existing approach cannot serve the purpose.

### Step 5 — Implement

Rules during implementation:

**Architecture**
- Place code where equivalent existing code lives.
- Follow the existing folder/naming convention exactly.
- Match the existing component, hook, and utility structure.
- Use existing type patterns; do not introduce new type conventions.

**Components**
- Use the project's existing component library before creating a new component.
- If creating a new component, follow the existing component structure (props interface, styling, export pattern).
- Do not replicate styling patterns that already exist in a shared component.

**State and data**
- Use the existing state management solution.
- Do not introduce a new state management library.
- Follow existing patterns for loading, error, and success state.
- Do not add global state for data that belongs at screen/component scope.

**API and data fetching**
- Use the existing API client / service layer.
- Follow existing error handling conventions for API calls.
- Do not invent a new HTTP client or data-fetching pattern.

**Navigation**
- Extend the existing navigation structure.
- Register new screens in the existing navigator(s), following existing patterns.
- Do not introduce a new navigation library or parallel navigator.

**Async operations**
- Follow the project's existing async pattern (async/await vs. Promise chains vs. observables).
- Handle loading, error, and cancellation in the same way as existing async operations.

**Forms**
- Use the project's existing form library or pattern if one exists.
- Do not introduce a new form library.

**Permissions**
- Check what permissions exist in `AndroidManifest.xml` and `Info.plist` before requesting new ones.
- Follow existing permission-request patterns.

**Platform-specific behavior**
- Use `.ios.ts`/`.android.ts` file splitting only when it is already established in the project.
- Do not introduce platform-splitting for small differences; use inline `Platform.OS` checks matching existing style.

**Accessibility**
- Apply `accessibilityLabel`, `accessibilityRole`, and related props in the same way as similar existing components.

**Styling**
- Use the existing theme/design-system tokens.
- Do not hardcode colors, font sizes, or spacing values that are available in the theme.

**Naming**
- Follow existing naming conventions exactly: casing, prefixes, suffixes.
- Do not rename existing things to satisfy a preferred naming style.

**Comments and documentation**
- Do not add explanatory comments unless the logic is non-obvious and no equivalent comment exists nearby.
- Do not add JSDoc to every function if the project does not use JSDoc.

### Step 6 — Testing

1. Identify whether the project has tests for similar features.
2. If yes, write tests that match the existing test structure and style.
3. If no tests exist for similar features, do not create tests unless explicitly asked.
4. Run the existing test suite (if available) to confirm no regressions.
5. Run type checking and linting if commands are available.

Do not introduce a new testing library.

### Step 7 — Review the implementation

Before reporting completion:
1. Re-read every file you created or modified.
2. Verify the implementation matches the requirement.
3. Verify all edge cases listed in the plan are handled.
4. Check for dead code (variables declared but unused, imports not used).
5. Check for accidental debug code (`console.log`, TODO comments, hardcoded test values).
6. Check that no unrelated files were modified.
7. Check that no unnecessary dependencies were added.

### Step 8 — Report

Summarize:
- What was implemented.
- Files created/modified.
- Anything deliberately left out and why.
- Any open questions or follow-up recommended.

---

## Handling specific scenarios

### Loading / error / empty states
Every async operation must have all three states handled. Find how the project currently renders these and match exactly.

### API integration
Locate the existing API service layer. Add new endpoints or methods there, following the existing method signature and error-handling pattern.

### Offline / caching behavior
If the project has offline or caching behavior, check whether the new feature needs to participate. Do not add caching where none exists unless the requirement specifies it.

### Authentication / authorization
If the feature requires an authenticated user, follow the existing auth guard / route protection pattern.

### Deep links
If the feature is accessible via a deep link, extend the existing deep-link handler; do not create a parallel one.

---

## Common mistakes to avoid

- Creating a new component that duplicates an existing one with a slightly different name.
- Adding a new state management pattern alongside the existing one.
- Hardcoding strings, colors, or sizes that should come from the theme or i18n layer.
- Forgetting to handle error and empty states.
- Adding dependencies without checking whether equivalent functionality exists already.
- Modifying files unrelated to the feature (e.g., touching shared utilities for the sake of "cleaning up").
- Writing tests that only test the happy path when the project's test patterns include negative cases.
- Creating platform-specific files where a simple conditional would suffice (or vice versa).

---

## Interaction with other skills

- **feature-analysis**: When available, read the feature analysis artifact before implementing. It provides the implementation plan, reuse map, and risk assessment so you do not need to re-derive them.
- **codebase-analysis**: Run first when entering a completely unfamiliar project or subsystem with no feature analysis artifact.
- **bug-fix**: If during implementation you discover an existing bug, note it — do not fix it inline unless it directly blocks the feature.
- **refactor**: Do not refactor unrelated code during feature implementation.
- **ui-ux-review**: Run after implementation to verify visual and UX quality against design references.
- **test-review**: Run after implementation to verify adequate test coverage.
- **code-review**: Run after implementation to catch correctness and architecture issues before reporting done.
- **project-context**: If CLAUDE.md is missing critical information needed during planning, update it via `project-context` after the feature is complete.
