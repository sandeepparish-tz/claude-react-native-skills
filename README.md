# Claude Code Skills — Universal Development Skills

A reusable set of Claude Code skills for production software projects. Install these into any project to give Claude Code consistent, evidence-first workflows for the full development lifecycle.

---

## Why these skills exist

Claude Code is powerful at writing code but, without explicit guidance, tends to:

- Create new implementations instead of reusing existing ones
- Invent architecture instead of following established patterns
- Fix symptoms rather than root causes
- Skip analysis and planning in favor of immediate coding
- Introduce unnecessary dependencies, abstractions, or complexity
- Make changes without verifying the result

These skills encode disciplined engineering workflows into Claude's behavior:

> **Inspect first. Don't assume. Reuse existing code. Make minimal changes. Verify everything.**

They help Claude:
- Understand the codebase before touching it
- Analyze requirements and identify reusable code before implementing
- Produce reviewable implementation plans before writing a line of code
- Debug systematically with evidence, not guesswork
- Refactor iteratively developed and AI-generated code safely
- Review UI/UX against design references and platform conventions
- Evaluate test coverage quality, not just coverage percentage
- Act as a final engineering quality gate before completion

---

## Skills

| Skill | Purpose | Example |
|-------|---------|---------|
| `project-context` | Analyze the project and create or update its `CLAUDE.md` | Initialize project context |
| `codebase-analysis` | End-to-end flow analysis from a defined start to end point, or broad orientation survey of a codebase | Trace the login flow end-to-end |
| `feature-analysis` | Analyze a feature requirement and produce a reviewable implementation artifact before coding | Analyze a feature before implementation |
| `feature-development` | Implement new functionality by reusing existing architecture, patterns, and utilities | Implement the notifications feature |
| `bug-fix` | Diagnose defects through evidence-based root-cause analysis; implement minimal, verified fixes | Diagnose and fix a reported crash |
| `refactor` | Safely improve code structure without changing behavior; effective on AI-generated or iterative code | Clean up a feature implementation |
| `ui-ux-review` | Inspect UI for visual consistency, UX quality, accessibility, and platform-specific behavior | Review a screen against a design |
| `code-review` | Final engineering quality gate: correctness, architecture, security, performance, and maintainability | Review the current implementation |
| `test-review` | Assess test coverage quality, identify meaningful gaps, and evaluate test reliability | Identify missing regression tests |
| `skill-customization` | Customize installed universal skills for project-specific architecture and conventions while preserving the universal workflow | Customize skills for this project |

---

## codebase-analysis: orientation and flow trace

`codebase-analysis` operates in two modes, selected automatically from your request.

**Orientation mode** surveys an unfamiliar codebase: architecture, technology stack, organizational patterns, conventions, and existing implementations. Use before starting feature work, bug investigation, or a large refactor in an unfamiliar area.

**Flow Trace mode** performs a deep execution trace of a specific path through the codebase from a defined start point to a defined end point. It traces the implementation through all layers encountered — event handlers, business logic, state updates, API calls, persistence, navigation, error paths, edge cases, and platform-specific behavior.

Example prompt:

```
Analyze the login flow from app launch to the Home screen.
Start at application launch and end when an authenticated user reaches Home.
Create the final flow analysis artifact.
```

The flow trace produces a structured artifact containing:

- Flow definition and scope
- Step-by-step execution path with ASCII diagram
- Per-step analysis: file, symbol, logic, conditions, state changes, calls, failure behavior
- Validation matrix
- Decision / branch matrix
- State flow
- Navigation flow
- API / data flow
- Error and failure paths
- Edge cases
- Platform-specific behavior (when applicable)
- Files and symbols involved
- Confirmed vs. inferred vs. unknown behavior
- Gaps, risks, and inconsistencies
- Verification scenarios
- Flow coverage summary

Flow Trace is not a file search or feature summary tool. It follows actual code references from the entry point to the end point.

---

## Project customization

Universal skills are designed to work immediately after installation. `skill-customization` makes them aware of the specific project they are operating in.

### Customization model

```
Universal Skill (SKILL.md universal content)
      +
Project-Specific Configuration section (appended below a separator)
      +
CLAUDE.md (project-wide context, read by all skills)
      =
Effective skill behavior for this project
```

Broad project conventions (architecture, commands, naming, constraints) belong in `CLAUDE.md` — all skills read it automatically. Information that improves exactly one skill's behavior (artifact storage location, specific test command, generated directories to exclude, design token file path) goes into that skill's `## Project-Specific Configuration` section.

The project section is appended after the universal content, clearly demarcated. Universal skill updates can be applied by replacing the universal content while preserving the project section below.

### Recommended first-time setup

```
project-context       ← create CLAUDE.md with project conventions
      ↓
skill-customization   ← add project-specific context to each relevant skill
```

### After universal skill updates

```
skill-customization   ← reapply project sections on top of updated universal skills
```

Example prompt:
```
The universal skills were updated. Update the installed skills while
preserving our project-specific customizations.
```

---

## Recommended workflows

These are suggested combinations, not mandatory pipelines. Each skill also works independently.

### Understand an existing flow

```
codebase-analysis      ← trace the flow from start point to end point
```

### Understand a new requirement before coding

```
feature-analysis       ← analyze requirements and produce an implementation artifact
```

### Analyze then implement

```
codebase-analysis      ← understand the relevant area
      ↓
feature-analysis       ← analyze requirements; produce and review artifact
      ↓
feature-development    ← implement using the artifact as a guide
```

### Full feature workflow

```
project-context        ← create or update CLAUDE.md if needed
      ↓
codebase-analysis      ← understand the relevant area
      ↓
feature-analysis       ← produce implementation artifact; review before coding
      ↓
feature-development    ← implement using the artifact as a plan
      ↓
ui-ux-review           ← verify visual and UX quality
      ↓
test-review            ← verify test coverage
      ↓
code-review            ← final quality gate
```

### Fix a bug

```
codebase-analysis      ← trace the affected flow (if area is unfamiliar)
      ↓
bug-fix                ← root cause → minimal fix
      ↓
test-review            ← confirm regression test was added
      ↓
code-review            ← verify the fix is clean
```

### Clean up an existing implementation

```
codebase-analysis      ← understand what actually exists
      ↓
refactor               ← remove duplication, dead code, patch-on-patch debt
      ↓
test-review            ← verify behavior is preserved
      ↓
code-review            ← verify the refactor is clean
```

### Set up a new project

```
project-context        ← create CLAUDE.md with discovered conventions
      ↓
skill-customization    ← adapt installed skills to the project
```

### Customize skills for a project

```
skill-customization    ← inspect project and apply project-specific customizations
```

---

## Skill categories

### Setup and configuration

These skills configure Claude's behavior for the project.

| Skill | Role |
|-------|------|
| `project-context` | Inspect project and create/update `CLAUDE.md` |
| `skill-customization` | Adapt installed skills to the project's architecture, tooling, and conventions |

### Analysis and review

These skills inspect, analyze, or evaluate without modifying application code (unless changes are explicitly requested).

| Skill | Role |
|-------|------|
| `codebase-analysis` | Orientation survey or end-to-end flow trace of existing implementation |
| `feature-analysis` | Analyze requirements and produce an implementation artifact before coding |
| `ui-ux-review` | Evaluate UI/UX quality, accessibility, and platform behavior |
| `code-review` | Final engineering quality gate before merge |
| `test-review` | Assess test coverage quality and identify gaps |

### Implementation and change

These skills write or modify application code.

| Skill | Role |
|-------|------|
| `feature-development` | Implement new functionality following existing architecture |
| `bug-fix` | Diagnose and fix a defect through root-cause analysis |
| `refactor` | Improve code structure without changing observable behavior |

The separation matters: analysis should precede implementation. Running `codebase-analysis` or `feature-analysis` before writing code is not optional overhead — it is the mechanism that prevents duplicated implementations, mismatched architecture, and wasted effort.

---

## Example prompts

Copy-paste ready prompts for every skill. Adjust the bracketed placeholders to match your task.

---

### Project context

```
Initialize the project context for this repository.

Inspect the project structure, existing CLAUDE.md, package and configuration
files, architecture, development commands, testing setup, and project conventions.

Create or update the project context without overwriting existing valid rules
or inventing project-specific information.
```

---

### Codebase analysis — end-to-end flow trace

```
Analyze the complete login flow in this application.

Start Point:
Application launch.

End Point:
A successfully authenticated user reaches the Home screen.

Trace the actual implementation from start to end, including
authentication and session checks, navigation, input validation,
state changes, API calls, persistence, success and failure paths,
and relevant edge cases.

Do not modify source code.

Create the final flow analysis artifact and clearly distinguish
confirmed behavior from inferred or unknown behavior.
```

### Codebase analysis — orientation survey

```
Survey this codebase before I start working on [feature/area].

Understand the project architecture, technology stack, folder structure,
existing patterns, state management, data layer, and navigation approach.
```

---

### Feature analysis

```
Analyze this feature request before making any code changes.

[Describe the feature or paste the requirement]

Inspect the existing implementation, architecture, UI patterns, state
management, API and data flow, navigation, and reusable components.

Create a reviewable implementation analysis artifact containing:
scope, existing behavior, required changes, affected areas, edge cases,
risks, testing considerations, and an implementation plan.

Do not modify application code until the analysis is complete.
```

---

### Feature development

```
Implement the following feature in this project.

[Describe the feature or reference the feature-analysis artifact]

First inspect the existing architecture and the closest related implementation.
Reuse existing patterns, components, and utilities where appropriate.

Do not introduce unnecessary dependencies, files, abstractions,
or architectural changes.

Implement the feature, update relevant tests, and verify the final
implementation against the requirements.
```

---

### Bug fix

```
Investigate and fix this bug.

Observed behavior:
[Describe what is happening]

Expected behavior:
[Describe what should happen]

Steps to reproduce:
[List reproduction steps, or paste a stack trace]

First trace the relevant execution flow through the existing codebase
to identify the root cause before making any changes.

Make the smallest appropriate fix. Avoid unrelated changes. Verify the
fix does not introduce regressions and run the relevant tests after the change.
```

---

### Refactor

```
Refactor the implementation of this feature.

[Describe the feature or code area to refactor]

First analyze the current implementation and identify unnecessary,
duplicated, patch-like, or unrelated code.

Preserve the existing behavior and requirements. Use the project's
existing architecture and patterns.

Remove unnecessary complexity without introducing unrelated changes.

After refactoring, verify the feature still works and run the relevant tests.
```

---

### UI/UX review

```
Review this screen or feature for UI and UX quality.

[Name the screen or feature]

First inspect the existing implementation and any available design reference files.

Compare the implementation against the project design system and provided
design references where available.

Review layout, spacing, typography, colors, components, interactions,
loading states, error states, empty states, accessibility, responsiveness,
and platform-specific behavior.

Do not redesign the feature or modify code automatically.

Report confirmed design mismatches, UX issues, accessibility issues,
engineering limitations, and subjective observations separately.
```

---

### Code review

```
Review the current implementation for this feature or change.

[Name the feature, describe the change, or specify the files to review]

Inspect the actual code and its surrounding context.

Look for correctness issues, regressions, architecture violations,
unnecessary complexity, duplicated logic, error-handling gaps,
performance concerns, security concerns, maintainability issues,
and unnecessary changes.

Prioritize real issues over stylistic preferences.

Do not modify the code. Provide actionable findings with relevant
file and line references.
```

---

### Test review

```
Review the testing strategy for this feature.

[Name the feature or describe what was recently changed]

First inspect the existing test setup and the actual implementation.

Identify missing tests, weak tests, brittle tests, duplicated tests,
incorrect mocks, missing error and edge case coverage, and important
user or business behaviors that are not verified.

Do not judge coverage percentage alone. Focus on whether the tests
provide meaningful protection against regressions.

Do not modify the tests unless explicitly requested.
```

---

### Skill customization

```
Customize the installed Claude Code skills for this project.

First inspect the project architecture, technology stack, CLAUDE.md,
existing Claude configuration, installed skills, development commands,
testing setup, folder structure, and project-specific conventions.

Identify which universal skills would benefit from project-specific customization.

Preserve the universal skill behavior and existing valid customizations.

Do not add temporary or task-specific rules.

Create a customization plan, then apply only safe and verified
project-specific customizations.
```

---

## Installation

### Tell Claude Code to install from this repository

In a Claude Code session inside your target project:

```
Install all skills from: <YOUR-GITHUB-REPOSITORY>

Inspect the repository, then install the skills into the current project's
.claude/skills/ directory. Do not overwrite skills that have already been
customized for this project.
```

Claude will follow the detailed instructions in [install/INSTALL.md](install/INSTALL.md).

### Install selected skills only

```
Install the following skills from: <YOUR-GITHUB-REPOSITORY>
- feature-analysis
- feature-development
- code-review
```

### Manual install

```bash
# From your target project root
mkdir -p .claude/skills

# Copy all skills
for skill in project-context codebase-analysis feature-analysis feature-development \
             bug-fix refactor ui-ux-review code-review test-review skill-customization; do
  cp -r path/to/.claude-skills/skills/$skill .claude/skills/
done
```

### After installation, generate CLAUDE.md

```
Run the project-context skill to analyze this project and create a CLAUDE.md.
```

This gives Claude a project-specific context file that all skills use to make better decisions.

---

## Expected project structure after installation

```
your-project/
├── CLAUDE.md                          ← created by project-context
└── .claude/
    └── skills/
        ├── project-context/
        │   └── SKILL.md
        ├── codebase-analysis/
        │   └── SKILL.md
        ├── feature-analysis/
        │   └── SKILL.md
        ├── feature-development/
        │   └── SKILL.md
        ├── bug-fix/
        │   └── SKILL.md
        ├── refactor/
        │   └── SKILL.md
        ├── ui-ux-review/
        │   └── SKILL.md
        ├── code-review/
        │   └── SKILL.md
        ├── test-review/
        │   └── SKILL.md
        └── skill-customization/
            └── SKILL.md
```

Project-specific conventions and constraints belong in `CLAUDE.md`, not in these universal skills.

---

## Updating skills

```
Update the installed skills from: <YOUR-GITHUB-REPOSITORY>

Compare each installed skill against the repository version.
Only update skills that have not been customized for this project.
Preserve any project-specific modifications.
```

Or manually, for a specific skill:

```bash
# Replace a single skill (only if it has not been project-customized)
cp -r path/to/.claude-skills/skills/code-review .claude/skills/code-review
```

---

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md` in this repository following the structure of the existing skills.
2. Add the skill entry to `skills.json`.
3. Update the skills table and workflows in this README.

Each skill should:
- Have a clear, single purpose.
- Define when to use it and when not to.
- Provide a specific, actionable workflow.
- Work independently as well as in combination with others.
- Not duplicate instructions already in another skill.
- Never assume a specific library, framework, or project structure.

---

## Repository structure

```
.claude-skills/
├── README.md
├── LICENSE
├── skills.json                    ← skill manifest (names, versions, paths)
├── skills/
│   ├── project-context/
│   │   └── SKILL.md
│   ├── codebase-analysis/
│   │   └── SKILL.md
│   ├── feature-analysis/
│   │   └── SKILL.md
│   ├── feature-development/
│   │   └── SKILL.md
│   ├── bug-fix/
│   │   └── SKILL.md
│   ├── refactor/
│   │   └── SKILL.md
│   ├── ui-ux-review/
│   │   └── SKILL.md
│   ├── code-review/
│   │   └── SKILL.md
│   ├── test-review/
│   │   └── SKILL.md
│   └── skill-customization/
│       └── SKILL.md
└── install/
    └── INSTALL.md
```

---

## Philosophy

**Inspect first.** Read `CLAUDE.md`, understand the codebase, find existing implementations before planning.

**Don't assume.** Confirm what the project actually uses — navigation library, state management, testing framework, UI system — by reading the code, not the package name.

**Reuse existing code.** Find the closest existing feature. Reuse its components, hooks, services, and patterns. Duplicate implementations are the most common failure mode.

**Make minimal changes.** Change only what the task requires. Do not refactor unrelated code. Do not add unrelated features. Do not introduce unnecessary dependencies.

**Verify everything.** Re-read every changed file before reporting done. Run type checking, linting, and tests when available.

---

## Universality

These skills are technology-agnostic. They discover and follow the conventions of whatever project they are used in.

They never assume:
- A specific language, framework, or runtime
- A specific navigation library
- A specific state management library
- A specific UI component library
- A specific testing library
- A specific folder structure
- A specific company, project name, or team convention

All project-specific decisions are discovered from the codebase or from `CLAUDE.md`.

Some skills include React Native–specific guidance (platform behavior, native modules, safe area handling) for projects where that is relevant. Those steps are skipped when the project is not React Native.

---

## License

MIT
