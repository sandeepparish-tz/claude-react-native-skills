# Skill: feature-analysis

## Description
Analyze a feature requirement completely and produce a structured implementation artifact before any code is written. This skill exists to separate thinking from doing: it surfaces ambiguities, maps the existing codebase, identifies reuse opportunities, and produces an implementation plan that can be reviewed before work begins.

**Do not modify application code during feature analysis unless explicitly requested.**

## When to use
- When starting a non-trivial feature where planning before coding prevents wasted effort.
- When requirements are ambiguous or complex enough to warrant review before implementation.
- When multiple engineers or stakeholders need to align before development begins.
- When running the recommended workflow: `feature-analysis` → review → `feature-development`.
- When asked to analyze, plan, or spec a feature without implementing it yet.

## When NOT to use
- For trivial changes (single-file edits, copy changes, config tweaks) — go directly to `feature-development`.
- When explicitly asked to just implement immediately.
- When a thorough analysis was already done earlier in the same session.

---

## Required workflow

### Step 1 — Understand the requirement

Read the requirement carefully and answer these questions before touching any code:

- What is the user's goal? What problem does this solve?
- What is in scope? What is explicitly out of scope?
- What does "done" look like?
- Are there design references (Figma, screenshots, wireframes)? If so, locate and read them.
- Are there API contracts, tickets, or specification documents? If so, read them.
- What assumptions are you making that could be wrong?

List ambiguities explicitly. Only ask about ones that would materially change the implementation. Do not ask questions that can be answered by reading the codebase.

### Step 2 — Inspect the project

Run a targeted inspection before planning anything:

1. Read `CLAUDE.md` if it exists.
2. Find the most similar existing feature in the codebase. Read it end-to-end.
3. Identify the organizational pattern: where would this feature's code live?
4. Identify relevant existing code:

```bash
# Find related screens, components, hooks, services
grep -r "relevant keyword" src/ --include="*.ts" --include="*.tsx" -l
find src/ -name "*RelatedName*" -type f
```

5. Read representative files to understand existing patterns for: state, data fetching, navigation, UI, error handling, forms, and testing.

### Step 3 — Map existing implementation

Document explicitly what already exists that is relevant to this feature:

**Screens / pages**: existing screens the feature lives on or navigates to/from.
**Components**: reusable components the feature can use.
**Hooks**: data-fetching or business-logic hooks to reuse.
**Services / API clients**: existing API integration the feature will call.
**Types / models**: existing data models the feature works with.
**Utilities**: shared utilities relevant to the feature.
**Navigation**: existing routes, navigators, and patterns.
**State**: existing state the feature reads or writes.

For each item found: name the file path, describe what it does, and note how it relates to the feature.

### Step 4 — Identify affected areas

List every part of the codebase the feature will touch:
- What existing files will be modified?
- What existing tests will need updating?
- What navigation structures will change?
- What shared state will be affected?
- What API endpoints need to exist or be extended?

### Step 5 — Analyze UI/UX (when applicable)

Only include this section when the feature has a UI component.

- Which screens are affected?
- What existing UI components will be reused?
- What states must the UI handle: loading, empty, error, success, disabled?
- What validation is needed and how is it currently done in the project?
- What are the interaction flows (tap, swipe, scroll, input, submit)?
- What are the accessibility requirements based on existing patterns?
- Are there Figma or design references? If yes, compare against the existing implementation pattern.
- Are there responsive or orientation considerations?
- Are there keyboard behavior / safe area considerations?

Do not invent design specifications. If design references are missing, identify that as an open question.

### Step 6 — Analyze API/data (when applicable)

Only include this section when the feature involves data.

- What existing API endpoints/methods are relevant?
- What new endpoints are needed? (Mark as dependency if backend contract is not yet defined.)
- What request and response shapes are expected? (Only document if contracts exist — do not invent them.)
- How will data be fetched: on mount, on demand, in background?
- How are errors handled in existing API calls? Follow the same pattern.
- Is pagination, caching, or optimistic updating relevant? Does the project already support it?
- What loading, error, and empty states must be handled?

If an API contract is missing, clearly mark it as a blocking dependency or open question.

### Step 7 — Analyze state management

- What existing state does the feature read?
- What new state will the feature introduce?
- Where does that state belong: global, screen, component?
- How does the project currently manage similar state? Follow the same approach.
- Does the feature cause state changes visible to other screens?

### Step 8 — Analyze navigation

- How does the user reach this feature? From where?
- How does the user leave? Back, dismiss, complete flow?
- Are new routes or screens needed?
- How are params passed (typed params, deep links, shared state)?
- Does the project use a specific navigator pattern? Follow it.

### Step 9 — Platform considerations (React Native)

Only include what is actually relevant to this feature:

- Does the feature behave differently on iOS vs. Android?
- Are there platform-specific permissions required? Check existing `AndroidManifest.xml` / `Info.plist` patterns.
- Are there keyboard, safe area, or hardware back-button considerations?
- Are there deep-link entry points?
- Do any native modules need to be involved?

### Step 10 — Identify risks and edge cases

**Edge cases**: what unusual inputs, states, or user behaviors must be handled?
**Regression risks**: what existing behavior could this feature accidentally break?
**Breaking changes**: does this change any shared API, type, or component contract?
**Platform differences**: any known iOS vs. Android behavioral differences?
**Performance**: any lists, images, or computations that could be expensive?
**Security**: any user data, auth tokens, or sensitive inputs involved?
**API dependencies**: is this feature blocked on an API that does not exist yet?
**State conflicts**: could this feature's state interfere with other active flows?

### Step 11 — Define the implementation plan

Produce a concrete, file-level plan. Only list files you have evidence to name. Do not invent file paths.

```
IMPLEMENTATION PLAN
-------------------

File/Area | Purpose | Change type | Dependencies | Notes
----------|---------|-------------|--------------|------
[path]    | [what]  | Create/Modify | [deps]     | [notes]
```

Also include:
- Reused: [component/hook/utility] from [path]
- New dependencies required: [none | package name — justification]
- Testing approach: [what to test, following existing patterns]

### Step 12 — Compile open questions

List only questions that are genuinely blocking:
- Missing API contracts.
- Missing design references.
- Ambiguous scope decisions that cannot be resolved from the codebase.
- Platform or permission decisions that require team input.

Do not ask about things that can be determined by reading the codebase or that are implementation details Claude can decide.

---

## Implementation artifact format

Produce the analysis as a structured artifact using only the sections relevant to the feature. Omit sections that do not apply.

```markdown
# Feature Analysis: [Feature Name]

## Objective
[1-2 sentences: what this feature does and why]

## Scope
**In scope**: [what this analysis covers]
**Out of scope**: [what is explicitly excluded]
**Assumptions**: [assumptions made during analysis]

## Existing Implementation
[Relevant screens, components, hooks, services, types found — with file paths]

## Architecture / Data Flow
[How data moves from source → state → UI for this feature]

## UI / UX
[Affected screens, reused components, states to handle, interaction flows]
[Only include if design references exist: design reference differences]

## API / Data
[Existing endpoints reused, new endpoints needed, request/response shapes if known]

## State Management
[What state is introduced, where it lives, how it follows existing patterns]

## Navigation
[Routes affected, params, entry/exit flows]

## Platform Considerations
[iOS/Android differences, permissions, keyboard, safe area — only what applies]

## Reusable Existing Code
| Item | Path | How used |
|------|------|----------|
| [component] | [path] | [usage] |

## Required Changes
| File | Change type | Purpose |
|------|-------------|---------|
| [path] | Create/Modify | [what changes] |

## Edge Cases
- [list]

## Risks
- [regression risk, breaking change, platform issue, etc.]

## Testing Strategy
[What to test, following the project's existing test patterns]

## Open Questions
- [blocking questions only]

## Implementation Plan
[Ordered sequence: what to do first, what depends on what]
```

Trim to what matters. A concise artifact covering the real decision points is more useful than a comprehensive document covering every possible angle.

---

## Common mistakes to avoid

- Inventing API contracts that don't exist yet (mark as open questions instead).
- Inventing design specifications when no design reference is provided.
- Listing files to create before confirming they don't already exist.
- Including sections that have nothing relevant to say for this specific feature.
- Over-documenting obvious decisions that any developer would make the same way.
- Starting implementation before the analysis is complete.

---

## Interaction with other skills

- **codebase-analysis**: If the project is entirely unfamiliar, run `codebase-analysis` first to establish orientation. `feature-analysis` then focuses on the specific feature area.
- **feature-development**: The output of this skill is the input to `feature-development`. When an analysis artifact exists, `feature-development` should read it and follow the implementation plan rather than re-deriving the same conclusions.
- **ui-ux-review**: After feature implementation, use `ui-ux-review` to verify the UI matches the design references and UX patterns identified here.
- **test-review**: The testing strategy section of the artifact informs `test-review`'s scope.
- **code-review**: The scope, edge cases, and risks sections of the artifact give `code-review` the context it needs to review the implementation effectively.
