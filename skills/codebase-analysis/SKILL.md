# Skill: codebase-analysis

## Description
Investigate a codebase before making meaningful changes, or trace the complete end-to-end execution of a specific flow between a defined start point and end point. Produces either a structured orientation report (broad survey) or a detailed flow-trace artifact (deep execution analysis). Read-only by default — does not modify source code.

## When to use

**Use Orientation mode when:**
- Entering an unfamiliar project before feature development, bug fixing, or refactoring.
- Asked to understand, map, or survey a codebase or subsystem.
- `CLAUDE.md` is absent, stale, or incomplete.
- A prior skill (feature-analysis, feature-development, bug-fix) needs project context first.

**Use Flow Trace mode when:**
- Asked to analyze a specific flow, journey, or path end-to-end (e.g., "analyze the login flow from app launch to home").
- Asked to trace what happens when a user performs a specific action.
- Debugging a non-obvious issue that spans multiple layers.
- Investigating whether a complete implementation path actually exists.
- Asked to produce a flow analysis artifact.

## When NOT to use
- When making a small, well-understood, self-contained edit — go directly to the implementation.
- When the task is documentation-only.
- When an adequate analysis was already completed earlier in the same session.
- When the goal is to plan a new feature — use `feature-analysis` instead.
- Do not run a full analysis when a targeted check would suffice.

---

## Step 0 — Determine the analysis mode

Read the user's request and select the appropriate mode before doing anything else.

**Orientation mode indicators**: "understand the codebase", "map the project", "survey the architecture", "what does this project use", "how is X organized", no specific start/end flow given.

**Flow Trace mode indicators**: a specific user action or system event as a start point ("from app launch", "when the user taps Buy", "from login screen"), a specific outcome as an end point ("until home screen", "until order confirmed", "until the file is uploaded"), or phrases like "trace the flow", "analyze what happens when", "end-to-end", "complete flow".

If the request is ambiguous: establish the most reasonable mode, state the assumption explicitly, and proceed. Do not ask unless the ambiguity would fundamentally change the analysis.

---

## Mode 1 — Orientation

Broad survey of the project's architecture, conventions, and existing implementations.

### Phase 1 — Orientation essentials (always run)

1. Read `CLAUDE.md` if it exists. Treat its content as authoritative until contradicted by evidence.
2. Read the root manifest file (e.g., `package.json`, `pubspec.yaml`, `Cargo.toml`, `go.mod`, `pom.xml`, or equivalent). Extract:
   - Project name, description
   - Scripts / tasks / commands
   - Dependencies — identify frameworks, routing, state management, API clients, UI libraries, testing tools, build tools
3. Inspect the root directory listing. Identify the overall shape: monorepo vs. single application, workspace layout, presence of platform-specific directories.
4. Read top-level config files present: language config (tsconfig, jsconfig), linting, formatting, testing, build, environment examples (never read `.env` with actual secrets).

### Phase 2 — Source structure

1. List the main source directory (`src/`, `app/`, `lib/`, `packages/`, or equivalent).
2. Identify the top-level organizational pattern: by feature, by type, by layer, or mixed.
3. For each major top-level folder, read one representative file to understand its purpose.
4. Identify:
   - Entry points (where the application starts)
   - Routing/navigation setup (library, route definitions, guards/redirects)
   - State management (library or pattern, where state is defined and accessed)
   - API/data layer (services, repositories, clients, data-fetching hooks/functions)
   - UI component organization (custom, third-party, or both)
   - Utility and helper organization
   - Type/interface definitions location
   - Platform-specific file conventions (if present)

### Phase 3 — Technology verification

Confirm (do not assume) by reading import statements and actual usage:
- Routing/navigation: which library or pattern, how routes are defined, how navigation is triggered.
- State management: which solution (or none), how state is created, read, and updated.
- API client: what HTTP client or SDK is used, base configuration, request/response patterns, error handling.
- Testing: which runner and assertion library, how tests are organized, what is mocked.
- Styling/theming: what approach, where theme/token values are defined.

### Phase 4 — Platform-specific specifics (when applicable)

Only run this phase when the project targets multiple platforms or specific runtime environments.

For each relevant platform found:
- Locate and read platform-specific configuration files (build configs, manifests, info files, entitlements).
- Identify platform-specific source file conventions.
- Note native modules, extensions, or plugins.
- Check for environment/variant configuration.
- Note deep-link or URL scheme configuration if present.

Skip this phase entirely if the project is a single-platform application with no platform-specific code.

### Phase 5 — Patterns and conventions

Read enough representative code to document:
- Naming: files, modules, components/views, functions, hooks/controllers, utilities, types, constants.
- Import style: absolute vs. relative, path aliases.
- Module/component structure: how a typical component/view/screen is organized.
- Error handling: how errors are caught, surfaced, and logged.
- Async patterns: async/await, Promises, observable streams, callbacks.
- Type usage: strictness, use of generics, any/unknown frequency.
- Comment conventions: sparse, JSDoc/doc comments, inline.

### Phase 6 — Existing implementations

Before finishing, search for:
- Any feature or flow the requested task resembles, to avoid duplication.
- Any utility, hook, service, or module the requested task might reuse.
- Any existing pattern the implementation must follow.

Use search rather than reading every file. Search for relevant keywords: screen names, function names, API route names, service names.

---

### Orientation output format

```
OBSERVED FACTS
- [Concrete facts derived directly from files — cite file path where important]

REASONABLE INFERENCES
- [Patterns extrapolated from observed code — flag clearly as inferences]

UNKNOWN
- [Things not found or not determinable without more evidence]

KEY PATTERNS
- [Conventions that any change in this codebase must follow]

CONSTRAINTS
- [Hard limits: do not change X, do not introduce Y, must use Z]

RISKY AREAS
- [Code that is fragile, not well-tested, or where changes have high blast radius]
```

Never present inferences as facts. Use language like "appears to," "likely," "based on X" for inferences.

### Orientation stopping criteria

Stop when you can answer:
1. What does this application do and who is it for?
2. Where would the requested task's code live?
3. What existing code would be reused or affected?
4. What patterns must any implementation follow?
5. What constraints must be respected?

Do not continue reading files once these are answered.

---

## Mode 2 — Flow Trace

Deep end-to-end execution analysis of a specific flow between a defined start point and end point. Read-only — does not modify any source code.

### Step 1 — Define flow boundaries

Establish and state explicitly before reading any code:

**Start Point**: The precise event, action, or condition that begins the flow (e.g., "application cold launch", "user taps the Submit button on the Checkout screen", "push notification received").

**End Point**: The condition that constitutes completion (e.g., "authenticated user is on the Home screen", "order confirmation screen is displayed", "file URL is returned to the caller").

**Scope**: What layers and systems are in scope (UI, state, API, persistence, navigation, platform, external SDKs). What is explicitly out of scope.

If the user's request implies but does not precisely state the boundaries, establish the most reasonable interpretation, declare it, and proceed. Ask only if the ambiguity would fundamentally change which code is traced.

### Step 2 — Orient to the relevant project area

Before tracing, establish a minimal project orientation scoped to the flow:

1. Read `CLAUDE.md` if it exists and has not already been read.
2. Read the root manifest to identify the technology stack.
3. Identify the application entry point — the file that runs first.
4. Identify the routing/navigation system — how screens/pages/views are registered and transitioned.
5. Identify the state management mechanism — how shared state is stored and updated.
6. Identify the API/data layer — how external calls are made.

This step is abbreviated if an Orientation analysis was already run in this session.

### Step 3 — Locate the entry point of the flow

Find where the flow actually begins in the codebase:

- If the start is application launch: find the root entry file, initialization code, and startup sequence.
- If the start is a user action: find the screen/view/component and the specific event handler.
- If the start is a system event (notification, deep link, background task): find the platform handler.

Read the entry point completely. Note: the exact trigger, the first function or handler called, and what the code does next.

Do not stop here — this is where the trace begins, not ends.

### Step 4 — Trace the execution chain

Follow the actual code relationships from the entry point toward the end point.

At each step, read the relevant code and follow its outward references:

```
Entry Point
  → identifies next function/handler/hook/service called
    → read that code
      → identifies next reference
        → follow it
          → continue until end point is reached or the chain cannot be followed
```

Trace through all relevant layers encountered — UI, event handlers, business logic, state updates, API calls, data transformation, persistence, navigation. The actual layers will vary by project; follow what the code actually does.

For each step in the chain, document:
- **What it is**: file path, symbol name (function, class, component, hook, method, etc.)
- **What it does**: the purpose of this step in the flow
- **What it calls next**: what it invokes or delegates to
- **Conditions**: any branches or guards at this step
- **Side effects**: state changes, API calls, persistence, events

Use search to find implementations that are not obvious from imports:
```bash
grep -r "functionOrSymbolName" src/ --include="*.ts" -l
grep -r "routeNameOrPath" src/ --include="*.ts" -l
```

### Step 5 — Branch and decision analysis

At every meaningful decision point found during the trace, document all branches:

For each decision:
- **Condition**: what is being evaluated
- **Source**: file and symbol where the decision happens
- **True path**: what happens when the condition is satisfied
- **False path**: what happens when it is not
- **Unhandled case**: what happens if the condition is absent or null

Do not invent branches that do not exist in the code. Do not omit branches that do exist.

### Step 6 — Validation analysis

For every significant input, data value, or precondition involved in the flow:

- **What** is validated
- **Where** — file and function
- **When** — on input, on submit, on API response, on startup
- **Valid condition** — what passes
- **Invalid condition** — what fails
- **Failure behavior** — error message, state update, navigation, retry allowed
- **Server-side validation** — if the project makes an API call, note whether the API also validates
- **Gaps** — any input that reaches an operation without apparent validation

Classify each validation finding as:
- **Confirmed**: directly observed in code
- **Inferred**: strongly implied by context but not directly seen
- **Unknown**: cannot be determined from available code

### Step 7 — State analysis

Trace every relevant state change in the flow:

For each state transition:
```
Previous state
  → Trigger (action, event, API response)
  → New state value
  → Where state is owned (global store, screen state, component state)
  → Who reads this state change (consumers)
  → Effect on the flow (navigation, UI update, side effect)
```

Identify:
- Initial state at flow start
- All reads of state during the flow
- All writes to state during the flow
- Whether state is persisted (survives restart)
- Loading, error, and success states
- Any state that should change but does not (gap)

Do not assume the project uses a specific state management library. Discover and describe what it actually uses.

### Step 8 — Navigation / routing analysis

For every navigation transition in the flow:

- **Trigger**: what causes the navigation (user action, state change, API response, condition)
- **Source**: file and symbol where navigation is called
- **Destination**: the target route, screen, page, or view
- **Conditions**: any guards or conditions that must be true for navigation to occur
- **Parameters passed**: what data is sent to the destination
- **Navigation type**: push, replace, reset, modal, redirect — using the project's actual terminology
- **Back behavior**: whether and how the user can return
- **Failure case**: what happens if navigation conditions are not satisfied

Do not assume the project uses a specific navigation library.

### Step 9 — API / external integration analysis

For every API call, SDK operation, or external service interaction in the flow:

```
UI / Handler
  → Service / Client
  → Request details (method, endpoint/operation, parameters, headers, auth)
  → Response shape
  → Success transformation and handling
  → Error handling (network failure, non-success status, malformed response)
  → State update after response
  → Navigation or UI change triggered by response
```

Only report what can be verified from the code. Note what is assumed or inferred separately.

### Step 10 — Persistence / storage analysis

For every read or write to local storage, cache, database, keychain, or file system:

- **What** is stored or read
- **Where** — which storage mechanism (identified from actual code)
- **When** — at which step in the flow
- **Format** — how data is serialized/deserialized if observable
- **Lifetime** — when it is cleared or expires
- **Effect on subsequent launches** — how persisted data changes behavior on the next app/session start
- **Restoration logic** — where this persisted data is read back and acted on

### Step 11 — Error and failure path analysis

For every significant operation in the flow, trace the failure behavior:

```
Failure trigger
  → Detection (where the error is caught or condition is checked)
  → Error handling (what the code does with the error)
  → User / system response (what the user sees or what state changes)
  → Recovery / retry (whether and how the user can retry)
  → Final state after failure
```

Cover failure categories relevant to the flow. Examples (only include what applies):
- Invalid or missing user input
- Authentication / authorization failure
- API error response
- Network unavailable
- Timeout
- Unexpected response shape
- Persistence read/write failure
- Missing required configuration
- Permission denied
- External SDK failure

Flag gaps: operations with no observable error handling.

### Step 12 — Edge case analysis

Identify edge cases relevant to this specific flow. Only include cases that are meaningful given the actual implementation.

For each edge case:
- **Scenario**: what unusual condition occurs
- **Handling**: implemented / partially handled / not handled / unknown
- **Evidence**: what in the code confirms or suggests this (for confirmed/inferred cases)

Examples of the kind of edge case to look for (not a mandatory list):
- Flow entered with an already-valid session or prior state
- Expired or invalid persisted credentials
- User navigates back mid-flow
- Duplicate submission (fast-tap, double-trigger)
- Empty or whitespace-only input
- Maximum-length input
- Network offline at various points in the flow
- API timeout
- Unexpected API response shape
- Application moved to background mid-flow
- Application restarted mid-flow

### Step 13 — Platform-specific behavior (when applicable)

Only run this step when the flow behaves differently across platforms.

If platform-specific files exist (e.g., `.ios.ts` / `.android.ts`, web vs. native variants), read both and document differences relevant to the flow.

Note any platform-specific:
- Permission handling
- Storage mechanisms
- Navigation behavior (hardware back button, swipe-back gesture)
- Keyboard behavior
- Deep link / URL scheme handling
- Native SDK behavior

Skip this step if the flow has no platform-specific implementation.

### Step 14 — Boundary verification

Explicitly verify that the traced flow connects the stated start point to the stated end point.

```
BOUNDARY VERIFICATION
---------------------
Start Point: [stated start]
End Point: [stated end]

✓ / ✗  [milestone 1 — file/symbol confirmed or not]
✓ / ✗  [milestone 2]
...
✓ / ✗  [end point confirmed or not]

Flow Trace Status: FULLY TRACED | PARTIALLY TRACED | BLOCKED / UNVERIFIED
```

If the trace cannot reach the end point:
- State exactly where it stops.
- Explain why (external SDK, missing code, dynamic dispatch, insufficient code access).
- Do not fabricate the missing steps.

---

### Flow trace artifact format

Produce the analysis as a structured artifact. Include only sections that are relevant to the flow. Omit sections with nothing substantive to report.

```markdown
# Flow Analysis: [Flow Name]

## 1. Flow Definition

**Objective**: [What is being analyzed and why]
**Start Point**: [Precisely where the flow begins]
**End Point**: [Precisely what constitutes completion]
**Scope**: [What is included / excluded]
**Assumed Boundaries**: [Any boundaries established by Claude, not explicitly stated by the user]

---

## 2. Executive Summary

[2-4 sentences describing how the flow currently works, its overall health, and any notable gaps or risks]

---

## 3. Complete Flow

[Step-by-step execution path. Use an ASCII diagram when it aids clarity.]

Example:
App Launch
  ↓
Initialize
  ↓
Check Persisted Session
  ↓
Session valid?
  ├── Yes → Navigate to Home
  └── No  → Navigate to Login
               ↓
           Render Login Form
               ↓
           User submits credentials
               ↓
           Validate input
           ├── Invalid → show inline error, remain on Login
           └── Valid
                  ↓
               Call authentication API
               ├── Error → show error, remain on Login
               └── Success
                      ↓
                  Persist session
                      ↓
                  Update auth state
                      ↓
                  Navigate to Home

---

## 4. Detailed Step Analysis

For each significant step:

### [Step Name]
- **File**: `path/to/file`
- **Symbol**: `FunctionName` / `ClassName.method` / `ComponentName`
- **Trigger**: what causes this step to execute
- **Logic**: what this step does
- **Conditions**: any branches
- **State changes**: what state is read or written
- **Calls**: what it delegates to next
- **Success**: what happens on the happy path
- **Failure**: what happens when this step fails
- **Evidence level**: Confirmed | Inferred | Unknown

---

## 5. Validation Matrix

| Input / Precondition | Where Validated | When | Valid Behavior | Invalid Behavior | Status |
|---------------------|-----------------|------|----------------|-----------------|--------|
| [field/value]       | [file:symbol]   | [on submit / on change / on mount] | [passes] | [error shown] | Confirmed / Inferred / Unknown |

---

## 6. Decision / Branch Matrix

| Decision Point | Condition | True Path | False Path | Source | Status |
|----------------|-----------|-----------|------------|--------|--------|
| [name]         | [what is checked] | [outcome] | [outcome] | [file:symbol] | Confirmed / Inferred |

---

## 7. State Flow

[Relevant state transitions throughout the flow]

```
[Initial state: description]
  → [Trigger]
  → [New state value]
  → [Owner: global / screen / component]
  → [Consumers of this state]
  → [Effect on flow]
```

---

## 8. Navigation Flow

| From | Trigger | Destination | Conditions | Params | Type | Status |
|------|---------|-------------|------------|--------|------|--------|
| [screen/view] | [event] | [target] | [guard] | [params] | [push/replace/reset] | Confirmed / Inferred |

---

## 9. API / Data Flow

[For each external call in the flow]

**[Operation name]**
- Source: `file:symbol`
- Endpoint / operation: `[method and path or SDK call]`
- Parameters: [request payload or arguments]
- Auth: [required / not required / from where]
- Success: [response shape and what happens]
- Failure: [error handling]
- Data transformation: [how response is mapped]
- Persistence: [whether result is cached or stored]

---

## 10. Error / Failure Paths

| Failure Scenario | Detection Point | Handling | User Response | Recovery | Status |
|-----------------|-----------------|----------|---------------|----------|--------|
| [scenario]      | [file:symbol]   | [how caught] | [what user sees] | [can retry?] | Confirmed / Inferred / Gap |

---

## 11. Edge Cases

| Scenario | Handled | Evidence | Notes |
|----------|---------|----------|-------|
| [edge case] | Yes / Partial / No / Unknown | [source or reasoning] | [notes] |

---

## 12. Platform-Specific Behavior

[Only include when platform differences exist in this flow]

| Behavior | Platform | Implementation | Source |
|----------|----------|----------------|--------|
| [behavior] | [platform] | [how it differs] | [file] |

---

## 13. Files and Symbols Involved

[Only files directly in the flow — not tangentially related files]

| File | Symbol | Responsibility | Flow Step |
|------|--------|----------------|-----------|
| `path/to/file` | `SymbolName` | [what it does] | [step name] |

---

## 14. Confirmed vs Inferred vs Unknown

### Confirmed
[Findings directly verified in source code, with file references]

### Inferred
[Reasonable conclusions based on surrounding implementation — not directly verified]

### Unknown
[Things that could not be established from the available code]

---

## 15. Gaps, Risks, and Inconsistencies

[Only include what is actually found — do not manufacture findings]

- **Missing handling**: [describe what operation has no error handling]
- **Broken transition**: [describe where a flow step does not connect to the next]
- **Missing validation**: [describe what input reaches an operation unvalidated]
- **Inconsistent state**: [describe where state could be out of sync]
- **Dead path**: [describe unreachable code found]
- **Duplicate logic**: [describe the same operation implemented in two places]

---

## 16. Verification Scenarios

[Scenarios a developer or tester can use to exercise the analyzed flow]

1. [scenario — what to do and what to expect]
2. ...

---

## 17. Flow Coverage

**Start Point**: [restated]
**End Point**: [restated]

**Primary Path**: Verified | Partially Verified | Unverified
**Validation Paths**: Verified | Partially Verified | Unverified
**Error Paths**: Verified | Partially Verified | Unverified
**Persistence**: Verified | Partially Verified | Not Applicable
**Navigation**: Verified | Partially Verified | Unverified

**Unknowns**:
- [what could not be verified and why]

**Flow Trace Status**: FULLY TRACED | PARTIALLY TRACED | BLOCKED / UNVERIFIED
```

---

### Evidence standards (both modes)

Apply to every significant finding:

**Confirmed**: Directly observed in source code. Cite the file path and symbol name.

**Inferred**: Reasonably concluded from surrounding implementation, but not directly read. State the basis for the inference.

**Unknown**: Cannot be established from the available code (external SDK, missing file, dynamic dispatch, insufficient access).

Never present inferences as facts. Use language like "appears to," "likely based on X," "this suggests" for inferences.

---

## Analysis is read-only by default

Do not modify application source code, configuration, tests, or dependencies during analysis. The purpose is to understand and document the existing implementation.

If the user requests changes after analysis, that is a separate task for the appropriate skill (feature-development, bug-fix, refactor).

---

## Common mistakes to avoid

**For both modes:**
- Presenting inferences as confirmed facts.
- Reading every file in the project instead of sampling or following references.
- Ignoring `CLAUDE.md` when it exists.
- Treating a missing file as confirmation a feature does not exist.

**For orientation mode:**
- Assuming the folder structure maps 1:1 to features — many projects mix organizational strategies.
- Assuming which library is used from the package name alone — confirm by reading import statements.
- Over-analyzing subsystems unrelated to the task.

**For flow-trace mode:**
- Stopping at the first screen involved in the flow without tracing the full execution chain.
- Analyzing only files whose names match the feature keyword rather than following actual code references.
- Inventing branches or behaviors not observed in the code.
- Declaring a flow "fully traced" without confirming the end point is reached.
- Omitting failure paths — tracing only the happy path.
- Listing every file in the project instead of only those that participate in the flow.

---

## Interaction with other skills

- **project-context**: If `CLAUDE.md` is absent or severely outdated after orientation, run `project-context` to create/update it.
- **feature-analysis**: `codebase-analysis` (flow trace) documents *existing* implementation. `feature-analysis` plans *new* implementation. Run flow trace first when the requirement involves extending or modifying an existing flow — the trace result becomes input to `feature-analysis`.
- **feature-development**: Run orientation mode before implementing in an unfamiliar area, unless a `feature-analysis` artifact already covers it.
- **bug-fix**: Run a scoped orientation or a targeted flow trace to understand the affected code path before diagnosing the root cause.
- **refactor**: Run orientation mode before a large-scale refactor to establish what is safe to change.
- **test-review**: Flow trace artifacts identify the verification scenarios and edge cases that `test-review` assesses coverage against.
- **code-review**: Orientation findings inform the project standards the review checks against. Flow trace artifacts provide the expected behavior the review validates the implementation against.
