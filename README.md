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

| Skill | Purpose |
|-------|---------|
| `project-context` | Analyze a project and create/update its `CLAUDE.md` with discovered conventions and constraints |
| `codebase-analysis` | Two modes: broad orientation survey of a codebase, or deep end-to-end flow trace from a defined start point to a defined end point |
| `feature-analysis` | Analyze a feature requirement and produce a reviewable implementation artifact before coding begins |
| `feature-development` | Implement new functionality by reusing existing architecture, patterns, and utilities |
| `bug-fix` | Diagnose defects through evidence-based root-cause analysis; implement minimal, verified fixes |
| `refactor` | Safely improve code structure without changing behavior; effective on AI-generated or iteratively patched code |
| `ui-ux-review` | Inspect UI for visual consistency, UX quality, accessibility, and platform-specific behavior |
| `code-review` | Final engineering quality gate: correctness, architecture, security, performance, and maintainability |
| `test-review` | Assess test coverage quality, identify important gaps, and recommend improvements |
| `skill-customization` | Adapt installed universal skills to the current project's architecture, tooling, and conventions |

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

### New feature (full workflow)
```
project-context        ← establish/update CLAUDE.md if needed
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

### Bug fix
```
codebase-analysis      ← if the subsystem is unfamiliar
      ↓
bug-fix                ← evidence → root cause → minimal fix
      ↓
test-review            ← confirm regression test was added
      ↓
code-review            ← verify the fix is clean
```

### AI-generated or messy codebase
```
codebase-analysis      ← understand what actually exists
      ↓
refactor               ← clean up patch-on-patch debt, dead code, duplications
      ↓
test-review            ← verify behavior is preserved
      ↓
code-review            ← verify the refactor is clean
```

### New project or stale context
```
project-context        ← create or update CLAUDE.md
      ↓
skill-customization    ← adapt installed skills to the project
      ↓
codebase-analysis      ← deep understanding of the full codebase
```

### Skills are also independently useful

Each skill can be invoked on its own. You do not need to run the full workflow for every task.

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

### End-to-end flow analysis

```
Analyze the login flow from app launch to Home.
Start at application launch and end when an authenticated user reaches Home.
Create the final flow analysis artifact.
```

### Codebase orientation

```
Survey this codebase before I start working on the notifications feature.
```

### Feature analysis

```
Analyze the checkout feature and create an implementation artifact before making any code changes.
```

### Bug investigation

```
Trace the execution flow related to this issue and identify the root cause before changing any code.
```

### Code review

```
Review the current implementation for correctness, architecture, and unnecessary changes.
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
             bug-fix refactor ui-ux-review code-review test-review; do
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
        └── test-review/
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
│   └── test-review/
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
