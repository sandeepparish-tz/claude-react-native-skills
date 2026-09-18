# Skill: skill-customization

## Description
Adapt installed universal Claude Code skills to the conventions, architecture, tooling, and constraints of the current project. Reads the project, determines what skill-specific guidance would materially improve each skill's behavior, and appends a clearly demarcated `## Project-Specific Configuration` section to the relevant installed skill files. Broad project conventions stay in `CLAUDE.md`; only genuinely skill-specific operating instructions go into individual skill files.

**Does not modify application source code. Does not replace universal skill content. Does not overwrite existing project customization without confirmation.**

## When to use
- After installing universal skills into a project for the first time — to adapt them to the project's actual architecture, tooling, and conventions.
- When the project has changed significantly (new framework, new state management, new test setup) and existing skill customizations are stale.
- After pulling updated universal skills from the source repository — to reapply project customization on top of the new universal content.
- When asked to "customize", "adapt", or "configure" the installed skills for this project.

## When NOT to use
- When no skills are installed in `.claude/skills/` yet — install them first via the installation workflow.
- When `CLAUDE.md` alone is sufficient — if the project conventions are fully captured there and the skills are behaving correctly, no further customization is needed.
- To store task-specific or temporary information (a particular bug's details, a one-off feature requirement, a deadline) — those belong in the conversation, not persisted to skill files.
- To rewrite universal skill workflow logic — customization adds project context, it does not replace the universal workflow.

---

## The customization model

There are two places where project-specific information belongs:

### `CLAUDE.md` — project-wide context

All installed skills read `CLAUDE.md` as their first step. Information that applies across the entire project and all skills belongs here:

- Project overview and purpose
- Technology stack and framework
- Architecture and folder structure
- Naming and coding conventions
- Development, build, and test commands
- General constraints ("do not introduce new dependencies", "all state through X")
- Generated directories that must not be modified

If information already exists in `CLAUDE.md`, do not duplicate it into individual skill files.

### `## Project-Specific Configuration` section — skill-specific context

Some information materially improves only one skill and is too narrow for `CLAUDE.md`. This goes into the installed `SKILL.md` as a clearly demarcated section appended after the universal content:

```
---

## Project-Specific Configuration

> Added by the `skill-customization` skill. Update by re-running this skill.
> To remove all project customization from this skill, delete this section.

[project-specific content]
```

The universal skill content above this separator is never modified. The section below it is owned by `skill-customization`.

This model means:
- Universal skill updates can be applied by replacing everything above the separator and re-appending the project section below.
- The boundary is explicit, self-documented, and can be inspected at any time.
- Individual skills remain fully functional if the project section is removed.

---

## Step 0 — Determine the operation mode

Read the user's request and select the appropriate mode before doing anything.

### Mode A — Initial customization
Indicators: skills are installed but contain no `## Project-Specific Configuration` section; user asks to "customize", "adapt", or "configure".

### Mode B — Update existing customization
Indicators: some skills already have a `## Project-Specific Configuration` section; user asks to "update" or "refresh" customization, or reports that project conventions have changed.

### Mode C — Reapply after universal skill update
Indicators: user says universal skills were updated, asks to "update installed skills while preserving project customization"; or the installed SKILL.md content differs from the universal version and contains a project section that needs to be preserved.

If the mode is ambiguous, assume Mode A for skills without a project section and Mode B for skills that already have one. State the assumption and proceed.

---

## Step 1 — Read existing project configuration

Before inspecting the project source code:

1. Find and read `CLAUDE.md` if it exists:
   ```bash
   find . -name "CLAUDE.md" -maxdepth 3
   ```
   Note every section and constraint already documented there. This is the primary source of project-specific truth — do not re-document its contents into skill files.

2. Find and inspect the installed skills:
   ```bash
   find .claude/skills -name "SKILL.md" | sort
   ```
   For each installed skill, check whether a `## Project-Specific Configuration` section already exists.

3. If any skill already has a project section, read it completely. Identify:
   - What is currently customized.
   - Whether any existing customization appears outdated or contradicts current project state.
   - Whether any customization is still valid.

4. Note the presence or absence of `.claude/agents/`, `.claude/settings.json`, or other Claude configuration files — do not modify them, but be aware of what Claude configuration already exists.

---

## Step 2 — Inspect the project

Run a targeted investigation to understand the project's actual architecture and tooling. Discover only what is verified. Do not assume.

### Always inspect:

**Root structure**
```bash
ls -la
```
Identify: monorepo vs. single app, presence of `package.json` / `pubspec.yaml` / `build.gradle` / `*.csproj` / `pyproject.toml` / `go.mod` / `Cargo.toml` / or equivalent.

**Primary manifest or dependency file** (read whichever is present):
- `package.json` — scripts, dependencies, devDependencies
- `pubspec.yaml` — dependencies, Flutter SDK version
- `build.gradle` / `settings.gradle` — Android/Gradle projects
- `pom.xml` — Java/Maven
- `pyproject.toml` / `setup.py` / `requirements.txt` — Python
- `go.mod` — Go
- `Cargo.toml` — Rust
- `*.csproj` / `*.sln` — .NET

Extract: language, framework, available commands, testing tools, linting tools.

**Source directory structure**:
```bash
ls src/ app/ lib/ 2>/dev/null || ls
```
Identify the top-level organizational pattern.

**Test setup** (read actual test config files and one representative test file):
- `jest.config.*`, `vitest.config.*`, `pytest.ini`, `build.gradle` test blocks, `pubspec.yaml` test deps
- Find where tests live: `find . -name "*.test.*" -o -name "*.spec.*" -o -name "*_test.*" | head -20`

**Lint / format configuration**: `.eslintrc.*`, `biome.json`, `ruff.toml`, `ktlint`, `.swiftlint.yml`, etc.

### Inspect conditionally (only when relevant):

**State management** — only if the project has UI or shared state:
- Search for: `grep -r "import.*store\|import.*zustand\|import.*redux\|Provider\|BlocProvider\|ChangeNotifierProvider\|StateNotifier\|GetX\|Riverpod" src/ --include="*.ts" --include="*.tsx" --include="*.dart" -l 2>/dev/null | head -10`

**Navigation / routing** — only if the project has screens or pages:
- Search for: `grep -r "NavigationContainer\|createBrowserRouter\|GoRouter\|Navigator\|Router\|useRouter" src/ --include="*.ts" --include="*.tsx" --include="*.dart" -l 2>/dev/null | head -10`

**UI / design system** — only if the project has UI:
- Look for: `theme`, `tokens`, `design-system`, `components` directories or files.

**API / data layer** — only if the project calls external services:
- Look for: `services/`, `api/`, `repositories/`, `data/`, HTTP clients, SDK initializations.

**CI/CD** — only if it affects skill behavior:
- `.github/workflows/`, `Makefile`, `Taskfile.yml`, `scripts/`
- Extract only commands that Claude would run (test, lint, build).

**Platform-specific directories** — only if the project targets multiple platforms:
- `android/`, `ios/`, `web/`, `macos/`, `windows/`, `linux/`

Stop investigating once you have enough evidence to make customization decisions. Do not read every file in the project.

---

## Step 3 — Classify what was discovered

For each discovered fact, classify it before deciding where it belongs:

### Universal (do not persist anywhere)
Facts that apply to all projects and are already encoded in the universal skills.
Example: "trace the complete execution flow from start to end point."

### Already in CLAUDE.md (do not duplicate)
Facts already documented in `CLAUDE.md`. Reference the existing documentation; do not copy it into skill files.

### Project-wide (belongs in CLAUDE.md if not already there)
Conventions that all skills benefit from equally.
Examples:
- The language and framework.
- Primary development commands.
- Folder structure overview.
- General naming conventions.
- Global constraints ("do not add dependencies without approval").

**If these are missing from CLAUDE.md:** Note them. The `project-context` skill is responsible for CLAUDE.md. If CLAUDE.md is absent or severely incomplete, recommend running `project-context` first. If customization is urgent and CLAUDE.md is absent, add only the most critical project-wide information to CLAUDE.md as a minimal stub — do not attempt to create a complete CLAUDE.md; that is `project-context`'s job.

### Skill-specific (belongs in the skill's project section)
Information that materially changes how exactly one skill should operate.
Examples:
- Test command to run after a bug fix (→ `bug-fix` and `test-review`).
- Where analysis artifacts should be stored (→ `codebase-analysis`, `feature-analysis`).
- Which design token file to check colors against (→ `ui-ux-review`).
- Which directories contain generated code that must not be refactored (→ `refactor`).

### Temporary / task-specific (do not persist)
Information relevant only to the current task or session.
Example: a specific bug being worked on, a deadline, a one-off feature requirement.
Do not write this to any file.

---

## Step 4 — Produce a customization plan

Before writing anything to disk, produce a reviewable plan.

If the scope is significant (multiple skills, material changes), present the plan as an artifact. For minor updates (one or two small additions), a brief inline summary is sufficient.

### Customization plan format

```
# Project Skill Customization Plan

## Project Overview
[Language, framework, architecture — one paragraph based on inspection]

## Existing Claude Configuration
[CLAUDE.md: present/absent/stale | Installed skills: list | Existing project sections: list]

## Classification Summary

### Already in CLAUDE.md (no action needed)
- [fact]: already documented

### Skill-Specific Findings

#### codebase-analysis
- Finding: [what was discovered]
- Recommended addition: [specific text to add]
- Reason: [why this materially improves the skill]
- Evidence: [file or symbol where this was confirmed]
- Action: Add to project section / Keep unchanged

#### feature-analysis
[same structure]

... [for each relevant skill]

## Skills with No Recommended Changes
- [skill]: [reason — e.g., nothing discovered that CLAUDE.md doesn't already cover]

## Conflicts / Ambiguities
[Anything that could not be cleanly resolved — describe what confirmation is needed]

## Risks
[Any existing customization that would be affected]
```

Trim the plan to what matters. Do not manufacture findings. Omit skills where no skill-specific customization is warranted.

---

## Step 5 — Apply safe customizations

After producing the plan, apply customizations that are:
- Directly verified from the project (not inferred or assumed).
- Not conflicting with existing customizations.
- Within the skill-specific scope (not duplicating CLAUDE.md).
- Not removing or weakening universal safeguards.

### For each skill to be customized:

1. Read the current installed `SKILL.md` completely.
2. Check whether a `## Project-Specific Configuration` section already exists.

**If no project section exists:**
Append the following to the end of the file:

```markdown
---

## Project-Specific Configuration

> Added by the `skill-customization` skill. Update by re-running this skill.
> To remove all project customization from this skill, delete this section.

[verified project-specific content]
```

**If a project section already exists (Mode B — update):**
Read the existing section carefully. Apply only incremental changes:
- Add newly discovered, verified information.
- Remove items that are demonstrably obsolete (e.g., a test command that no longer exists).
- Update items that are incorrect based on current project evidence.
- Preserve items that are still valid even if not re-discovered this session — they may reflect team decisions not obvious from code.
- Do not rewrite the entire section; preserve existing valid content.

### Ask for confirmation before applying:

- Replacing or removing any existing project section content.
- Adding a convention that is inferred rather than directly verified.
- Adding any rule that changes a previously established project constraint.
- Adding opinionated behavior not clearly supported by project evidence.

---

## Step 6 — Handle Mode C: universal skill update with project customization

When universal skills are updated and project customizations must be preserved:

1. For each installed skill that has a `## Project-Specific Configuration` section:
   a. Read the entire current SKILL.md (universal content + project section).
   b. Extract and save the project section (everything from `## Project-Specific Configuration` to end of file).
   c. Replace the skill file with the new universal SKILL.md content.
   d. Append the saved project section to the end of the new universal content.

2. For skills without a project section: simply replace with the new universal content.

3. After replacement, review each re-appended project section against the new universal content:
   - Check whether any project section item now conflicts with the updated universal content.
   - Check whether any project section item is now redundant with new universal content (the universal skill may have incorporated similar guidance).
   - Flag conflicts and redundancies in the report — do not silently discard them.

4. Do not claim to merge or version-diff universal content. The model is: preserve the project section as-is; apply the new universal content. Conflicts are surfaced in the report.

---

## Step 7 — Verify the result

After applying all customizations:

1. Re-read every file that was modified.

2. Verify for each modified skill:
   - [ ] Universal content is completely intact above the separator.
   - [ ] The separator `---` and the `## Project-Specific Configuration` heading are present.
   - [ ] The advisory comment is present (explains how to update/remove the section).
   - [ ] No universal safeguards were weakened or removed.
   - [ ] No technology assumptions were introduced without project evidence.
   - [ ] No task-specific or temporary information was persisted.
   - [ ] No information already in `CLAUDE.md` was duplicated.
   - [ ] The skill still functions as a standalone workflow if the project section were removed.

3. Verify `CLAUDE.md` if it was modified:
   - [ ] Existing content was not deleted.
   - [ ] Only additions were made, not rewrites.
   - [ ] No contradictions with existing content.

4. Verify no unrelated files were modified.

---

## Step 8 — Report

```
SKILL CUSTOMIZATION REPORT
===========================

Mode: [Initial / Update / Post-universal-update]

Project Context Discovered
--------------------------
[Language, framework, architecture — one short paragraph]

Skills Reviewed
---------------
[All installed skills that were inspected]

Skills Customized
-----------------
[Only skills where the SKILL.md was changed — list with one-line reason]

Skills Left Unchanged
---------------------
[Skills inspected but not changed — with one-line reason]

CLAUDE.md Changes
-----------------
[Added / Not modified — with reason]

Files Changed
-------------
[Exact file paths]

Existing Customizations Preserved
----------------------------------
[Project section items that were kept from a prior run]

Conflicts / Open Questions
--------------------------
[Anything that needs user input or could not be resolved]

Final Status
------------
Universal skills: Preserved
Project customization: Applied
Existing customizations: Preserved / Partially updated (describe)
Open conflicts: None / [list]
```

---

## Per-skill customization guidance

Use this as a checklist during Step 3. Only add what is actually discovered and verified.

### `codebase-analysis`

Look for:
- The project's source root and top-level organizational pattern (feature-first, layer-first, monorepo workspace layout).
- Entry points (where the application starts).
- Where analysis or flow-trace artifacts should be stored, if a convention exists (`docs/`, `analysis/`, inline, none).
- Directories that are generated and should be excluded from analysis (e.g., build outputs, `.dart_tool`, `node_modules`, generated API clients).

Do NOT add: navigation library names, state management library names — those belong in CLAUDE.md.

### `feature-analysis`

Look for:
- Where implementation artifacts should be stored, if a project convention exists.
- Any established design/Figma workflow (e.g., design tokens file location, Figma naming convention).
- Any required artifact sections mandated by the team.

Do NOT add: the full architecture — CLAUDE.md covers that.

### `feature-development`

Look for:
- Architecture rules that are not obvious from code (e.g., "all API calls must go through the repository layer, never directly from components").
- Specific forbidden patterns (e.g., "do not use `any` — use unknown and narrow", "never call platform APIs directly from business logic").
- Required patterns for new code (e.g., "all new screens must include an error boundary matching `src/components/ErrorBoundary`").

Do NOT add: folder structure, framework name, state management library — those belong in CLAUDE.md.

### `bug-fix`

Look for:
- Error monitoring or crash reporting tool and how to read its output (e.g., Sentry, Crashlytics, Bugsnag — only if confirmed present).
- Debug commands specific to this project's setup (e.g., `adb logcat` filters, specific log tags, `flutter logs`, specific gradle tasks).
- Platform-specific reproduction requirements (e.g., must test on both iOS and Android simulators before closing a bug).

Do NOT add: general debugging advice — that is universal.

### `refactor`

Look for:
- Directories that are generated and must not be touched (codegen output, migration files, proto-generated code).
- Architecture boundaries that must not be crossed (e.g., "domain layer must not import from infrastructure layer").
- Established abstractions that should be aligned with, not replaced.

Do NOT add: general refactoring principles — those are universal.

### `code-review`

Look for:
- Security requirements specific to the project (e.g., "all user data must be encrypted at rest using the project's `SecureStore` wrapper").
- Performance benchmarks or requirements if explicitly defined.
- Dependency policy (e.g., "all new dependencies require approval from architecture review").
- CI gates that must pass (if the project documents them and Claude can verify them locally).

Do NOT add: generic code quality principles — those are universal.

### `test-review`

Look for:
- The actual test runner and command (verified from `package.json` scripts, `Makefile`, `pubspec.yaml`, etc.).
- Where tests live (alongside source, `__tests__/`, `test/`, etc.) — verified by actual file discovery.
- The mocking approach (MSW, jest.mock, Mockito, Mocktail, etc.) — verified from actual test files.
- End-to-end test setup if present (Detox, Maestro, Appium, Espresso, XCTest UI, etc.).
- Coverage threshold if defined in config (do not invent one).
- Required test command to run after changes (verified from CI config or project documentation).

Do NOT add: general test quality principles — those are universal.

### `ui-ux-review`

Look for:
- The project's theme or design token file location (verified by finding the actual file).
- The component library in use (verified by reading import statements, not inferred from package name).
- Figma file naming convention or workflow if documented.
- Accessibility requirements that are explicitly established in the project (e.g., WCAG level, existing `accessibilityLabel` pattern).
- Whether dark mode is supported (verified by finding dark theme implementation).
- Platform targets (iOS, Android, web, desktop) — only what is confirmed.

Do NOT add: generic UX principles or accessibility guidelines — those are universal.

### `project-context`

Rarely needs skill-specific customization. `project-context` creates `CLAUDE.md`. Only add a project section if the project has a non-standard `CLAUDE.md` location or a required structure mandate.

---

## What NOT to customize

Do not add any of the following to project sections:

- **Temporary task information**: "For the current sprint, we are using feature flag X."
- **One-off bug details**: "This bug is caused by a race condition in component Y."
- **Individual developer preferences**: "Developer A prefers flat Redux reducers."
- **Ticket references**: "See JIRA-1234 for context."
- **Unverified assumptions**: "This project probably uses Redux." (not confirmed)
- **Information already in CLAUDE.md**: duplicates create contradictions over time.
- **Generic programming advice**: "Always handle errors." (already in universal skills)
- **Restatements of universal behavior**: "Inspect before acting." (already universal)

---

## Evidence standards

Every item written to a project section must be classified:

**Confirmed**: Directly observed in a source file, config file, or script. Cite the file path.

**Inferred**: Strongly implied by surrounding evidence but not directly read. State the basis. Only add inferred items when they are high-confidence and material.

**Unknown**: Do not add unknown items. If something cannot be confirmed, note it in the report as an open question.

---

## Common mistakes to avoid

- Copying CLAUDE.md contents into every skill file — this creates duplicates that diverge and contradict each other over time.
- Adding technology-specific guidance that was not verified (e.g., writing "use Redux dispatch" when Redux was not confirmed).
- Writing project sections that are so long they obscure the universal content.
- Adding task-specific information that will be wrong in the next session.
- Removing existing project section content just because it was not re-discovered in the current inspection — it may reflect team decisions not visible in code.
- Weakening universal safeguards (e.g., adding "skip analysis when the area seems familiar" to a skill that requires analysis).
- Treating `CLAUDE.md` as optional — if it is absent, recommend creating it via `project-context` before doing detailed skill customization.
- Customizing skills that were not installed — check that `.claude/skills/<skill-name>/SKILL.md` exists before modifying it.

---

## Interaction with other skills

- **project-context**: Run `project-context` first if `CLAUDE.md` is absent or severely outdated. `skill-customization` reads `CLAUDE.md` as its primary source of project truth — the quality of `CLAUDE.md` directly affects customization quality.
- **codebase-analysis**: If the project's architecture is unclear and `CLAUDE.md` is incomplete, run `codebase-analysis` (Orientation mode) before customizing skills — its findings provide the evidence base for customization.
- **All other skills**: Those skills read `CLAUDE.md` and their own project section. `skill-customization` is the only skill that writes to other skill files. After running it, the other skills should operate with better project-specific context without any further intervention.
