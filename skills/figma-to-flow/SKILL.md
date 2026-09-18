# Skill: figma-to-flow

## Description
Convert a Figma design — a single screen, a section, a set of related frames, or a full Figma page — into a verified, implementation-ready static UI and navigation flow specification. Uses the Figma MCP to inspect the actual design source. Inspects the existing project architecture, navigation patterns, and reusable components before producing the specification. Separates static UI and navigation implementation from API and backend integration, which is deferred by default.

**Does not modify application source code. Produces a specification artifact. Does not implement the feature unless explicitly asked.**

## When to use
- When a Figma design exists and needs to be turned into an implementation plan before coding begins.
- When a Figma page contains multiple screens and the screen relationships and navigation flow need to be mapped out.
- Before implementing a new screen, flow, or feature whose design lives in Figma.
- When static UI and navigation should be implemented first, with API/data integration deferred.
- When the implementation team needs a reviewable specification that separates what is confirmed, inferred, and unknown before touching the codebase.

## When NOT to use
- To review an **existing implementation** against a Figma design — use `ui-ux-review`.
- To analyze a **general feature requirement** (not design-driven) — use `feature-analysis`.
- To **implement** the feature — use `feature-development` after the specification has been reviewed.
- To map existing code end-to-end — use `codebase-analysis`.
- When no Figma design exists and the requirement is text-based only.

---

## Default behavior

**Read-only.** Do not create or modify any application source files. Produce a specification artifact only.

If the user explicitly asks to proceed with implementation after reviewing the specification, recommend the `feature-development` skill rather than implementing from within this skill.

**Static UI first.** The default scope of the specification is static UI and navigation. API, backend, and data integration are documented as future integration points, not immediate implementation tasks, unless the user explicitly scopes them in.

**Figma MCP first.** When the Figma MCP is available, use it as the primary source of truth for design details. Do not rely on screenshots, descriptions, or assumptions when MCP data is accessible.

---

## Required workflow

### Step 1 — Establish context and scope

Before inspecting anything:

1. What was provided? Identify the input:
   - Figma URL (file, page, section, frame, or node)
   - User description of what to convert
   - Whether this is a single screen or a multi-screen flow
2. Read `CLAUDE.md` if it exists. Treat its content as authoritative.
3. Determine whether other existing skills should be consulted first:
   - If the project architecture is unfamiliar: note that `codebase-analysis` (orientation mode) would be useful; run a targeted version if necessary rather than a full orientation.
   - If there is an existing `feature-analysis` artifact for the same feature: read it before proceeding — it may already cover parts of the required analysis.
4. State the assumed scope explicitly before proceeding. If the scope is ambiguous, establish the most reasonable interpretation and proceed — ask only if the ambiguity would materially change the output.

---

### Step 2 — Inspect the Figma design using Figma MCP

Use the Figma MCP (`mcp__figma__get_figma_data`) to inspect the provided Figma source.

**Do not skip this step when MCP is available.** The actual Figma source is the source of truth for visual details, component structure, and screen relationships. Do not rely on user descriptions or screenshots when MCP data is accessible.

Inspect and extract:

**Structure**
- Top-level frames and their names
- Sections and groupings
- Screen hierarchy and nesting
- Page contents (for multi-screen pages)

**Per screen / frame**
- Screen name and purpose
- Layout structure (header, body, footer, scrollable regions, fixed elements)
- UI components present: buttons, inputs, tabs, cards, lists, icons, images, navigation bars, bottom sheets, modals, badges, chips, steppers, progress indicators
- Component variants and states (selected, disabled, active, focused, empty, error, loading)
- Typography: font families, sizes, weights, line heights — as used in the design
- Colors: backgrounds, text, borders, icons, interactive states
- Spacing and sizing: padding, margin, gap, dimensions where visible
- Images and assets: illustrations, photos, icons, placeholders
- Text content: labels, placeholders, error messages, headings, body copy

**Interactions and connections (where available)**
- Prototype connections between frames
- Tap/click targets
- Navigation triggers (button presses, tab switches, back actions)
- Modal and bottom-sheet triggers
- Overlay and drawer behaviors

**If Figma MCP is unavailable or cannot access the requested resource:**
- Note the limitation clearly in the specification.
- Proceed using what information is available (user description, screenshots, or URL inspection).
- Mark all visual and structural details derived without MCP as `[Inferred — Figma MCP unavailable]`.
- Do not fabricate design details.
- Ask the user for the missing information only when it materially affects the navigation or implementation.

---

### Step 3 — Identify screens and multi-screen relationships

Whether the input is a single screen or a full page, complete this step.

**Single screen input:**
- Identify the one screen, its purpose, and its role in the broader application.
- Identify UI states within the screen (e.g., empty, populated, loading, error, selected).
- Identify interactive elements and their intended outcomes.

**Multi-screen input (page, section, or multiple frames):**

1. List all frames/screens found on the page.
2. Determine which screens belong to the requested feature or flow — not every screen on a page is necessarily in scope.
3. Identify the intended entry point (first screen in the flow).
4. Map the screen relationships:
   - Primary navigation path (linear flow A → B → C)
   - Branching paths (action on A leads to B or D depending on condition)
   - Modal and overlay flows (screen that opens above the current screen)
   - Back navigation
   - Alternate states (same screen with different content)
   - Error and empty states
5. Build a navigation map:

```
Entry Screen
   │
   ├── Action A → Screen B
   │                │
   │                ├── Action → Screen C
   │                └── Action → Modal D
   │
   ├── Action B → Screen E (replace)
   │
   └── Tab switch → Screen F
```

6. Note screens on the Figma page that are visually present but appear **out of scope** for the current flow.
7. If the intended relationships between screens cannot be determined from the Figma design or prototype connections, list them as open questions.

---

### Step 4 — Inspect the existing project architecture

Do not treat this as an isolated design-to-code task. Before forming any implementation recommendation, inspect the existing application.

**Navigation**
- What navigation library or pattern is used? (Discover from code — do not assume.)
- What existing routes and screens exist?
- How are new screens registered?
- What navigation patterns are established: stack, tab, drawer, modal, bottom sheet?
- How are navigation params passed?
- How is back behavior handled?

**Reusable UI components**
- What component library or design system exists?
- What existing components match elements in the Figma design: buttons, inputs, cards, lists, modals, bottom sheets, tabs, navigation bars, icons?
- What existing screen templates or layouts are reusable?

**Design system / theming**
- Where are typography tokens defined?
- Where are color tokens defined?
- Where is spacing defined?
- What icon set is used?
- What image/asset loading pattern is used?

**UI patterns**
- How are loading, error, and empty states implemented in existing screens?
- How are modals and bottom sheets presented?
- How are forms and inputs structured?
- How is static/local state managed in existing screens?
- What is the existing pattern for selected/unselected states?

**Existing screens related to the Figma flow**
- Do any of the Figma screens correspond to screens that already exist in the codebase?
- Are these new screens or redesigns of existing ones?

Reuse existing patterns wherever they apply. Do not propose a new architecture, library, or pattern simply because the Figma design looks different from the current implementation.

---

### Step 5 — Analyze every interactive element

For every meaningful interactive element identified in the Figma design:

- **What is it?** (button, tab, link, input, card, icon, back control, swipe gesture)
- **What triggers it?** (tap, long press, swipe, focus, change)
- **What is the intended outcome?**
  - Navigate to another screen
  - Open a modal or bottom sheet
  - Close/dismiss something
  - Change local UI state (select, expand, collapse, toggle)
  - Submit a form
  - Filter or sort content
  - Load more content
  - Execute a destructive action (requires confirmation?)
- **Is the outcome static?** Can this interaction be implemented without API data?
- **If an API is required:** document as a future integration point, not part of the static implementation scope.

Classify each action's outcome as:
- **Static** — implementable without any API or external data
- **Requires data** — needs real data but may be stubbed statically for now
- **API-dependent** — requires a live API call to function in production (defer)

If the intended behavior cannot be determined from Figma or from the existing application, mark it as **Unknown** and list it as an open question.

---

### Step 6 — Ask only necessary questions

Before finalizing the specification, identify whether any open questions would materially affect:

- Which screen is the entry point
- Whether a button navigates to an existing screen or a new screen
- Whether modal behavior is dismissible
- Whether a flow is linear or conditional
- Whether a Figma screen replaces an existing screen or adds a new one
- Whether specific UI state should persist across navigation

**Do not ask** questions that can be answered by reading the Figma design or the codebase.

**Do not ask** about implementation details that Claude can reasonably decide from existing patterns.

Group all questions together in a single block rather than interrupting the workflow.

If no questions are blocking, proceed directly to the specification artifact.

---

### Step 7 — Classify all behavior

Every significant behavior, interaction outcome, or design decision must be classified:

**Confirmed** — directly supported by Figma MCP data, existing code, or an explicit user requirement.

**Inferred** — reasonably concluded from existing application patterns, adjacent screens, Figma prototype connections, or standard UI conventions in the project.

**Unknown** — cannot be determined with confidence. Never silently convert an unknown into an implementation decision.

---

### Step 8 — Create the specification artifact

Produce the specification using the format below. Include only sections that are relevant to the feature. Omit sections that have nothing substantive to report.

---

```markdown
# Figma-to-Flow Specification: [Feature / Module Name]

## Overview

**Purpose**: [1-2 sentences describing the feature and its user goal]

**Figma Source**
- URL / reference: [Figma URL or reference provided]
- Page / section / frame: [What was inspected]
- Relevant screens: [List of frame/screen names inspected]

**Figma MCP Status**: Available and inspected | Unavailable — details inferred from [source]

---

## Scope

**In scope**: [What this specification covers]
**Out of scope**: [What is explicitly excluded — including API/backend unless scoped in]
**Static implementation boundary**: [What will be implemented now vs. deferred]

---

## Screen Inventory

For each screen in the flow:

### [Screen Name]

| Property | Value |
|----------|-------|
| Purpose | [What this screen does] |
| Entry point | Yes / No — reached from [screen] via [action] |
| Layout | [Header, body, footer, scrollable region, fixed elements] |

**UI Elements**
- [List each significant UI element with its type and purpose]

**Interactive Elements**
| Element | Action | Outcome | Static? | Status |
|---------|--------|---------|---------|--------|
| [element] | [tap/swipe/etc.] | [navigate / open / state change] | Yes / No / Partial | Confirmed / Inferred / Unknown |

**UI States**
| State | Trigger | Description | Status |
|-------|---------|-------------|--------|
| [state name] | [what causes it] | [how it looks] | Confirmed / Inferred / Unknown |

**Design Details** (from Figma MCP)
- Typography: [relevant type styles]
- Colors: [relevant colors]
- Spacing: [key dimensions]
- Assets: [images, icons]

---

## Navigation Flow

[ASCII diagram of screen-to-screen navigation]

```
[Entry Screen]
   │
   ├── [Action] → [Screen B]
   │                 │
   │                 └── [Action] → [Screen C]
   │
   ├── [Action] → [Modal D]
   │
   └── [Back] → [Previous Screen / App Exit]
```

**Navigation detail**

| From | Action / Trigger | To | Type | Params | Back behavior | Status |
|------|------------------|----|------|--------|---------------|--------|
| [screen] | [action] | [screen] | push / replace / modal / tab | [params if any] | [back behavior] | Confirmed / Inferred / Unknown |

---

## Static Implementation Scope

What should be implemented in the first pass:

- [ ] Render each screen with static/hardcoded or local-state data
- [ ] Navigation between screens as mapped above
- [ ] Tab switching (if applicable)
- [ ] Modal and bottom-sheet presentation and dismissal
- [ ] Selected / unselected / active states
- [ ] Form fields and static validation (where represented in the design)
- [ ] Loading and error states using static placeholders (where represented in the design)
- [ ] Back navigation behavior
- [ ] Other: [any feature-specific static behaviors]

---

## Deferred Integration

What should NOT be implemented yet:

| Item | Reason for deferral | Notes |
|------|---------------------|-------|
| [API endpoint] | Not required for static UI | Implement when backend is ready |
| [Authentication] | Not required for static flow | Wire in when auth is stable |
| [Real data fetching] | Static placeholder sufficient | Replace with real calls later |
| [Server-side validation] | Client static validation only for now | |

---

## Existing Code Reuse

| Existing item | Type | Location | How used in this flow |
|---------------|------|----------|----------------------|
| [ComponentName] | Screen / Component / Hook / Utility | [file path] | [usage] |

**Existing patterns to follow:**
- Navigation: [navigation pattern to follow]
- Modal/bottom-sheet: [modal pattern to follow]
- Loading/error/empty state: [pattern to follow]
- Form/input: [form pattern to follow]

---

## Implementation Plan

Ordered steps for implementing the static UI and navigation flow:

1. [First step — e.g., register new screens in navigator]
2. [Second step — e.g., create Screen A with static layout]
3. [Continue for each screen and navigation connection]
4. [Wire interactive elements to navigation]
5. [Implement UI state changes (tabs, selected, expand)]
6. [Add modal / bottom-sheet presentation]
7. [Implement static form behavior]

---

## Open Questions

[Only questions that materially affect navigation, screen behavior, scope, or implementation approach and cannot be answered from Figma or the codebase]

1. [Question — what it affects — options]
2. ...

If none: *No open questions. Proceed to implementation.*

---

## Risks and Ambiguities

| Item | Type | Description | Recommendation |
|------|------|-------------|----------------|
| [item] | Ambiguous Figma behavior / Missing design state / Architecture conflict | [description] | [how to handle] |

---

## Skills Considered

| Skill | Role | Decision |
|-------|------|----------|
| `codebase-analysis` | Understand existing architecture | [Used / Recommended before implementation / Not needed — project already familiar] |
| `feature-analysis` | Analyze feature requirements | [Used existing artifact / Not applicable — design-driven, not requirement-driven] |
| `ui-ux-review` | Review implementation against design | Recommended after `feature-development` completes |
| `feature-development` | Implement the feature | Recommended next step after specification review |

---

## Figma MCP and Design Access Notes

[Note any limitations: frames not accessible, prototype connections not available, design details inferred rather than confirmed, etc.]
```

---

### Step 9 — Verify before finalizing

Before producing the final artifact, verify all of the following:

**Figma**
- [ ] Relevant Figma frames were inspected via MCP (or limitation was noted).
- [ ] All screens belonging to the requested flow were identified.
- [ ] Each screen's interactive elements were reviewed.
- [ ] Every important interaction has a defined outcome.
- [ ] Design details came from MCP data, not invented assumptions.

**Codebase**
- [ ] Existing navigation structure was inspected.
- [ ] Existing reusable components relevant to the Figma elements were identified.
- [ ] Existing project conventions were applied to the specification.
- [ ] Screens that already exist in the codebase were identified.

**Flow**
- [ ] Entry point is defined.
- [ ] All screen-to-screen transitions are mapped.
- [ ] Back behavior is defined for each screen.
- [ ] Modal and overlay flows are defined where applicable.
- [ ] All interactive elements have defined outcomes.
- [ ] Unknown behavior is explicitly labeled — not silently decided.

**Scope**
- [ ] Static UI scope is clearly separated from API/backend scope.
- [ ] No API implementation is included unless the user explicitly requested it.
- [ ] No unnecessary architectural changes are proposed.

**Artifact**
- [ ] All behaviors are classified as Confirmed, Inferred, or Unknown.
- [ ] Open questions are listed (and are genuinely blocking, not cosmetic).
- [ ] Implementation plan is actionable by a developer who has not seen this conversation.

---

## Skill output summary

At the end of the workflow, provide:

1. **Analysis summary** — brief description of what was found in Figma and the project.
2. **Screens identified** — names and count.
3. **Navigation flow** — ASCII diagram or short description.
4. **Open questions** — if any; otherwise confirm none.
5. **Static implementation scope** — what is covered now.
6. **Deferred scope** — what is not in scope (APIs, backend, data binding).
7. **Specification artifact** — location or inline.
8. **Skills considered** — which existing skills were used or recommended.
9. **Figma MCP status** — whether MCP was used and any limitations.

---

## Common mistakes to avoid

- Relying on the user's description of the Figma design instead of inspecting MCP data directly.
- Treating every frame on a Figma page as part of the requested flow.
- Inventing design details (colors, spacing, component behavior) when MCP data is unavailable — mark as inferred or ask.
- Automatically including API and backend implementation when only static UI was requested.
- Proposing a new navigation library or architecture because the Figma design looks unfamiliar.
- Silently converting an unknown interaction into a specific implementation decision.
- Asking questions that can be answered by reading the Figma design or the codebase.
- Modifying source code during specification.
- Producing a generic design summary instead of an implementation-oriented specification.
- Assuming a Figma frame has the same navigation behavior across projects without checking the existing codebase.

---

## Interaction with other skills

- **codebase-analysis**: Run orientation mode before `figma-to-flow` when the project is entirely unfamiliar. `figma-to-flow` performs a targeted inspection by default, but a full orientation survey gives it better context for reuse decisions and navigation patterns.
- **feature-analysis**: If a `feature-analysis` artifact already exists for the same feature, read it before proceeding — it may contain scope decisions, API contracts, or reuse findings that this specification should align with. `figma-to-flow` focuses on the design source and static UI; `feature-analysis` focuses on requirements and full-stack planning. They are complementary, not duplicative.
- **ui-ux-review**: `ui-ux-review` validates whether a completed implementation matches the design. `figma-to-flow` produces the specification used to guide that implementation. Run `ui-ux-review` after `feature-development` completes.
- **feature-development**: The specification artifact produced by `figma-to-flow` is the direct input to `feature-development`. When implementing, read this artifact and follow its implementation plan, reuse map, and navigation flow rather than re-deriving them.
- **skill-customization**: Project-specific Figma conventions (where design assets are stored, the design token file path, the project's component library name, the MCP configuration) can be added to this skill's `## Project-Specific Configuration` section via `skill-customization`.
