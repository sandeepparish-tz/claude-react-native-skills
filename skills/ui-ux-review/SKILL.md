# Skill: ui-ux-review

## Description
Inspect completed UI implementation for visual consistency, layout correctness, interaction quality, UX flow, accessibility, and platform-specific behavior. Produces a severity-classified report. Does not redesign the application and does not make code changes unless explicitly requested.

## When to use
- After implementing a feature with UI changes, before reporting done.
- When asked to review the UI, UX, or visual quality of a screen or flow.
- As part of the recommended workflow after `feature-development`.
- When a design reference (Figma, screenshot, spec) exists and needs to be compared against the implementation.

## When NOT to use
- To review code correctness or architecture — use `code-review`.
- To review test coverage — use `test-review`.
- To redesign screens or propose new design directions without being asked.
- For backend, API, or non-visual logic changes.

---

## Default behavior

**Do not make code changes.** Produce a review report. Only apply fixes if the user explicitly requests it.

**Do not treat subjective preferences as defects.** Distinguish clearly between design mismatches, UX issues, accessibility issues, engineering limitations, and personal preferences.

---

## Required workflow

### Step 1 — Establish context

Before reviewing anything:
1. What is being reviewed? (specific screen, flow, or set of changes)
2. Are there design references available? (Figma links, screenshots, wireframes, design tokens, design system docs)
   - If yes: read them and use them as the source of truth for design decisions.
   - If no: evaluate against the project's own existing patterns and conventions.
3. Is there a CLAUDE.md that documents the UI system, theme, or design conventions?
4. What platform(s) are being reviewed? (iOS, Android, both)

### Step 2 — Understand the project's UI baseline

Read enough existing screens/components to understand what "correct" looks like for this project:
- How are spacing, typography, and colors applied? (theme tokens, StyleSheet, inline)
- What is the existing pattern for loading, error, and empty states?
- What interaction patterns are standard? (buttons, modals, forms, confirmations)
- Are there accessibility patterns already established?

Do not evaluate the implementation against an invented design standard. Evaluate it against:
1. Design references (highest priority, if available).
2. Existing established patterns in the project.
3. Platform UI guidelines (where clearly relevant and not contradicted by the project's design).

### Step 3 — Review the implementation

Work through each dimension below. Skip dimensions that are clearly not applicable.

---

## Review dimensions

### Visual consistency

**Spacing**
- Is spacing consistent with the project's spacing system?
- Are there arbitrary pixel values where spacing tokens should be used?
- Is spacing between elements visually balanced?

**Typography**
- Are font sizes, weights, and families consistent with the theme?
- Are there hardcoded font values where theme tokens should be used?
- Is text truncation handled correctly for long strings?

**Colors**
- Are colors sourced from the theme or design tokens?
- Are there hardcoded color values that should come from the theme?
- Is color contrast sufficient for text readability?
- Is dark mode handled if the project supports it?

**Iconography and imagery**
- Are icons from the project's established icon set?
- Are icon sizes consistent with similar usage elsewhere?
- Are images displayed at appropriate resolution?

**Sizing and alignment**
- Are component dimensions consistent with similar components?
- Are elements visually aligned (left edge, center, baseline)?
- Are borders, radii, and shadows consistent with the design system?

**Component consistency**
- Do new UI elements visually match similar existing elements?
- Are custom components needed, or could existing shared components be used?

---

### Layout and responsive behavior

**Screen sizes**
- Does the layout work on small screens as well as large?
- Are there elements that overflow or get clipped on narrow screens?

**Safe areas**
- Are safe area insets respected (top notch, bottom home indicator, navigation bar)?
- Is content pushed correctly when the keyboard appears?

**Scrolling**
- Is scrollable content actually scrollable?
- Does the scroll view extend to the correct boundaries?
- Is pull-to-refresh implemented correctly if the project uses it?

**Overflow and dynamic content**
- What happens when text is very long?
- What happens when a list is very long?
- What happens when data is absent vs. present?

**Orientation**
- If the project supports landscape orientation, does the layout hold up?

---

### Interaction quality

**Touch targets**
- Are tappable elements large enough? (Minimum 44×44pt is a common platform guideline — apply if relevant to the project.)
- Is there sufficient spacing between adjacent touch targets?

**State feedback**
- Do buttons show pressed/active state?
- Do interactive elements indicate when they are disabled?
- Is there visual feedback when an action is processing?

**Loading states**
- Is there a loading indicator while data is fetching?
- Does the loading state match the project's established pattern?
- Is the UI stable when the loading state transitions to loaded?

**Disabled states**
- Are disabled controls visually distinct?
- Are they correctly non-interactive when disabled?

**Navigation and back behavior**
- Does back navigation work as expected on both platforms?
- Are swipe-to-dismiss or hardware back button behaviors handled?

**Destructive actions**
- Are destructive actions (delete, clear, logout) confirmed before executing?
- Does the confirmation match the project's established pattern?

---

### UX quality

**Empty states**
- Is there a meaningful empty state when a list or content area has no data?
- Does the empty state match the project's established pattern?

**Error states**
- Are errors surfaced to the user with a meaningful message?
- Is there a way to retry after an error?
- Does the error state match the project's established pattern?

**Validation and form feedback**
- Are validation errors shown inline, clearly linked to the relevant field?
- Is validation timing correct (on submit, on blur, or as-you-type — matching project patterns)?
- Are required fields indicated?

**Flow and discoverability**
- Can the user accomplish the primary goal without confusion?
- Are there unnecessary steps in the flow?
- Are calls-to-action clear and prominent?
- Is it clear what happens next after each action?

**Confirmation and feedback**
- Is the user informed when an action completes successfully?
- Does the success feedback match the project's established pattern (toast, banner, navigation)?

---

### Accessibility

Evaluate accessibility against what the project has established. Do not impose accessibility requirements not present in similar existing screens, but do flag clear omissions.

**Labels**
- Do interactive elements have `accessibilityLabel` or equivalent?
- Are labels descriptive enough to be useful for screen reader users?

**Roles**
- Are `accessibilityRole` values set on custom interactive elements (buttons, checkboxes, etc.)?
- Do complex components communicate their role correctly?

**State**
- Are `accessibilityState` values set correctly (checked, disabled, selected, expanded)?

**Touch target size**
- Are touch targets large enough for users with motor impairments?

**Color contrast**
- Is text contrast sufficient? (Reference platform accessibility guidelines if no project standard exists.)

**Dynamic text**
- Does the layout hold up when the user increases system font size?
- Is text truncated correctly rather than overflowing its container?

**Focus order**
- Is the logical reading/interaction order correct for screen readers?

---

### React Native platform behavior

**Android vs. iOS differences**
- Are there behaviors that differ between platforms and are handled correctly?
- Does status bar styling work correctly on both?
- Are Android elevation/shadow differences handled?
- Is the back button (hardware back) handled correctly on Android?
- Does iOS swipe-back gesture work correctly?

**Keyboard behavior**
- Does the keyboard obscure content that the user needs to interact with?
- Is `KeyboardAvoidingView` or an equivalent used correctly?
- Does the keyboard dismiss correctly when expected?

**Safe area**
- Are `SafeAreaView` or equivalent insets applied where needed?
- Is the bottom of the screen clear of the home indicator / navigation bar?

**Native controls**
- Are platform-native controls used where appropriate (pickers, date inputs)?
- Do they follow platform conventions?

**Permissions**
- If a feature requires permissions, is the permission request handled gracefully?
- Is there an appropriate degraded state when permissions are denied?

---

## Design reference comparison (when references exist)

When Figma links, screenshots, or design specs are provided:

1. Compare the implementation against the reference for each visible screen state.
2. Document differences by category:
   - **Design mismatch**: The implementation differs from the design reference (spacing, color, component, layout).
   - **UX issue**: The implementation or design creates a usability problem.
   - **Accessibility issue**: A required accessibility attribute is missing or incorrect.
   - **Engineering limitation**: The design cannot be implemented as specified due to platform or library constraints (note, do not automatically fix).
   - **Subjective preference**: The implementation looks different from the design but both are acceptable — flag only if relevant.

Do not mark a deviation from the design as a defect if it is an intentional engineering adaptation. Confirm before classifying.

---

## Findings classification

| Severity | Meaning |
|----------|---------|
| **Critical** | Broken interaction, invisible content, crash on interaction, complete mismatch with design reference on a key screen. |
| **High** | Missing required state (no loading, no error), broken navigation, significant design deviation, missing accessibility label on a primary action. |
| **Medium** | Inconsistent spacing/color/typography, minor design deviation, UX friction in a non-critical flow. |
| **Low** | Minor visual inconsistency with limited user impact. |
| **Suggestion** | Improvement worth considering; no obligation to act on it. |

---

## Review report format

```
UI/UX REVIEW REPORT
===================

Screen / Flow:
[What was reviewed]

Reference:
[Design reference used, or "project patterns" if none provided]

SUMMARY
-------
[2-3 sentences: overall quality, main concerns]

CRITICAL ISSUES
---------------
[CRITICAL] [Category] — [Screen/Component]
Issue: [Description]
Evidence: [What was observed]
Impact: [Why it matters]
Action: [Recommended fix]

HIGH PRIORITY
-------------
[Same format]

MEDIUM PRIORITY
---------------
[Same format]

LOW PRIORITY / SUGGESTIONS
---------------------------
[Same format]

ACCESSIBILITY
-------------
[Accessibility-specific findings, or "No accessibility issues found"]

PLATFORM-SPECIFIC ISSUES
-------------------------
[iOS / Android specific findings, or "No platform-specific issues found"]

DESIGN REFERENCE DIFFERENCES
------------------------------
[Only if a design reference was provided]
[List of deviations with classification: mismatch / limitation / preference]

RECOMMENDATIONS
---------------
[Summary of must-fix vs. should-fix vs. consider actions]
```

---

## Common mistakes to avoid

- Flagging design choices as defects when they match the project's established patterns.
- Applying platform UI guidelines that contradict the project's intentional design decisions.
- Inventing design requirements when no reference was provided.
- Making code changes without being asked.
- Reviewing code logic instead of UI/UX behavior.
- Treating every visual difference from a design reference as Critical — assess actual user impact.
- Ignoring the difference between a design mismatch (factual) and a subjective preference.

---

## Interaction with other skills

- **feature-analysis**: The UI/UX analysis section of `feature-analysis` identifies design references and UI requirements up front. `ui-ux-review` validates whether the implementation matches them.
- **feature-development**: Run `ui-ux-review` after feature implementation to catch visual and UX issues.
- **code-review**: `ui-ux-review` covers presentation layer quality; `code-review` covers code correctness and architecture. Run both independently — they are complementary.
- **test-review**: Accessibility findings from this skill may indicate tests (e.g., `accessibilityLabel` presence) that should be added.
