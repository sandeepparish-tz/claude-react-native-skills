# Skill: project-context

## Description
Analyze an existing project and create or update its `CLAUDE.md` — the project-specific context file that guides all future Claude Code interactions. Preserves valid existing content, removes only what is demonstrably obsolete, and documents only what is observed, not assumed.

## When to use
- When CLAUDE.md does not exist for a project.
- When CLAUDE.md is outdated and no longer reflects the current codebase.
- When the project architecture has materially changed.
- When a team adopts Claude Code for the first time on an existing project.
- After running `codebase-analysis` and identifying that the existing context is incomplete.

## When NOT to use
- When CLAUDE.md is recent and accurately reflects the current project.
- When making routine changes that do not require a context update.

---

## Required workflow

### Step 1 — Find and read existing CLAUDE.md

```
find . -name "CLAUDE.md" -maxdepth 3
```

If found:
1. Read it completely.
2. Note its sections, claims, and commands.
3. Mark each claim as: still valid, needs update, unknown/unverified, or obsolete.

**Do not overwrite it until analysis is complete. Do not discard valid content.**

If not found, proceed with a blank slate.

### Step 2 — Analyze the project

Run a structured investigation matching the scope of `codebase-analysis`. At minimum cover:

**Repository shape**
- Root directory listing
- `package.json` (root + any workspace packages): name, version, description, scripts, dependencies, devDependencies
- Monorepo vs. single app
- Presence of `android/`, `ios/`, `web/`, `packages/`, `apps/`

**Source structure**
- Main source directory
- Organizational pattern (by feature, by type, by layer, mixed)
- Entry points

**Technology stack** (read code to confirm, do not infer from package.json alone)
- Language: TypeScript, JavaScript, or mixed
- Framework: React Native, React, Expo, bare React Native, etc.
- Package manager: npm, yarn, pnpm, bun (check lockfile)
- Navigation library and pattern (read actual route definitions)
- State management (read actual store/context/hook implementations)
- API/data layer (read actual service/client code)
- UI component library (read actual component imports)
- Form handling (if used)
- Testing framework (read actual test files)
- Build tooling

**React Native specifics (when applicable)**
- `android/app/build.gradle`: applicationId, minSdkVersion, targetSdkVersion, build types, flavors
- `ios/Podfile`: Pod dependencies, platform version
- Environment configuration: `.env.*` files structure, config library used
- Native modules: custom Java/Kotlin or Objective-C/Swift files

**Code conventions** (read 5-10 representative files)
- Naming: files, components, hooks, utilities, types, constants
- Import style: absolute vs. relative, path aliases
- Component structure: props interface pattern, export pattern
- Error handling pattern
- Async pattern

**Development commands** (from `package.json` scripts only — do not invent)
- Start / run
- Build
- Test
- Lint
- Type check

**Constraints** (from existing documentation, CLAUDE.md, or clearly established patterns)
- What must not be changed
- What must be used
- What must not be introduced

### Step 3 — Identify conflicts with existing CLAUDE.md

For each claim in the existing CLAUDE.md, determine:
- **Still accurate**: Preserve as-is.
- **Partially accurate**: Update to reflect current state.
- **Obsolete**: Remove only if there is clear evidence it no longer applies.
- **Unverifiable**: Keep with a note if it contains important constraints, or remove if it is factual and contradicted by evidence.

Do not remove information just because you did not observe it during analysis — it may reflect intentional project decisions or constraints not evident from code alone.

### Step 4 — Draft CLAUDE.md

Follow the structure below. Omit sections that do not apply. Do not fabricate content for sections with nothing to document.

---

## CLAUDE.md structure

```markdown
# [Project Name]

## Project overview
[1-3 sentences: what the app does, who it is for, what platforms it runs on]

## Architecture

### Folder structure
[Brief description of main source directory and top-level organization]
[Only describe the actual structure — do not idealize it]

### Key patterns
[How features are organized: screens, components, hooks, services, etc.]
[What the data flow looks like: source → state → UI]

## Technology

- **Language**: [TypeScript / JavaScript / mixed]
- **Framework**: [React Native / Expo managed / Expo bare / React Native CLI]
- **Package manager**: [npm / yarn / pnpm / bun]
- **Navigation**: [library name + brief note on pattern used]
- **State management**: [library/pattern + brief note]
- **API layer**: [approach: custom fetch wrapper / axios instance / React Query / SWR / etc.]
- **UI**: [custom only / design system library / mix]
- **Testing**: [Jest + Testing Library / Detox / none observed]

## Development commands

```bash
# [Verified from package.json scripts]
[command]     # [purpose]
```

## Code conventions

### Naming
- [File naming: kebab-case / PascalCase / camelCase]
- [Component files: PascalCase]
- [Hook files: useCamelCase]
- [Utility files: camelCase]

### Imports
- [Absolute paths via alias @/ / relative only / mixed]

### Components
- [Props interface pattern]
- [Export pattern: default / named]

### Error handling
- [How errors are caught and surfaced]

### Async
- [async/await / Promise chains / observable]

## React Native

### Android
- Application ID: [from build.gradle, if found]
- Min SDK: [from build.gradle, if found]
- Target SDK: [from build.gradle, if found]
- Build types: [debug/release/staging, if found]

### iOS
- Deployment target: [from Podfile, if found]
- Key pods: [notable native dependencies]

### Environment
- [How environment variables are managed: react-native-config / dotenv / etc.]
- [.env file naming convention: .env.development / .env.staging / etc.]

## Constraints

[Rules Claude must follow in this project. Only include rules supported by evidence.]

Examples of the kind of rule to include:
- Use [existing component] for [purpose] — do not create a new one.
- All colors must come from [theme/tokens path] — do not hardcode values.
- Do not introduce a new [library category] — the project uses [existing solution].
- State for [specific domain] must go through [specific store/pattern].
- [Specific folder] is generated — do not edit by hand.

## Notes

[Anything that does not fit above but is important for Claude to know:
unusual project decisions, known technical debt areas, areas under active development,
important third-party integrations, deployment considerations, etc.]
```

---

## CLAUDE.md quality rules

**Include:**
- Specific, actionable facts about this project.
- Rules that prevent incorrect implementation decisions.
- Verified commands (only from `package.json` scripts).
- Constraints that are not obvious from the code.

**Exclude:**
- Generic programming advice that applies to all projects.
- Obvious facts (e.g., "React components are functions").
- Speculative information ("probably uses," "might have").
- Duplicate information already evident from the project structure.
- Long explanations of how frameworks work.
- Every command from `package.json` — only the ones Claude will regularly use.

**Language:**
- Use direct, factual language for confirmed information.
- Use "observed" or "appears to" for inferences.
- Do not present inferences as project policy.

### Length guidance
CLAUDE.md should be concise enough to read in under two minutes. If it exceeds that, it will not be read carefully by Claude in future sessions. Prioritize high-value, non-obvious information.

---

## Merging with existing CLAUDE.md

When an existing CLAUDE.md is present:

1. Do not replace it wholesale. Merge findings.
2. Preserve project-specific constraints and rules, even if you did not independently observe them — they may reflect team decisions not expressed in code.
3. Update factual sections (commands, dependencies, SDK versions) when the evidence clearly contradicts them.
4. Add newly discovered sections that were missing.
5. Remove sections only when they contain demonstrably false information.

---

## When to recommend a CLAUDE.md update

After completing major work, recommend updating CLAUDE.md if:
- A new library was added to the project.
- A new architectural pattern was established.
- Build or environment configuration changed.
- A constraint was added or removed.
- The folder structure changed significantly.

---

## Interaction with other skills

- **codebase-analysis**: This skill performs a focused version of codebase-analysis specifically to produce CLAUDE.md. Run codebase-analysis first if a broader understanding of the codebase is also needed.
- **feature-analysis**: CLAUDE.md constraints and conventions inform feature analysis — run `project-context` first on a new project before doing feature analysis.
- **feature-development**: CLAUDE.md directly improves feature development by providing established patterns and constraints upfront.
- **bug-fix**: CLAUDE.md constraints and commands reduce the chance of introducing a fix that violates project conventions.
- **refactor**: CLAUDE.md documents established patterns, helping refactor align with intended architecture.
- **ui-ux-review**: CLAUDE.md documents the UI system and theme conventions that `ui-ux-review` uses as a baseline.
- **test-review**: CLAUDE.md documents the testing framework and approach that `test-review` discovers automatically — keeping it up-to-date reduces repeated discovery.
- **code-review**: CLAUDE.md provides the project standards the review checks against.
