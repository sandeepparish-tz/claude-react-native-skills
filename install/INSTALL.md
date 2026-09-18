# Installing Skills into a Target Project

This document tells Claude Code how to install, update, or selectively install skills from this repository into an existing project.

---

## How Claude Code skills work

Claude Code loads skills from `.claude/skills/<skill-name>/SKILL.md` within a project. Each skill is a Markdown instruction file that defines how Claude should behave for a specific type of task.

This repository provides ten universal skills:

| Name | Purpose |
|------|---------|
| `project-context` | Create/update the project's CLAUDE.md |
| `codebase-analysis` | Map and understand the codebase |
| `feature-analysis` | Analyze requirements and produce an implementation artifact |
| `feature-development` | Implement features following existing patterns |
| `bug-fix` | Diagnose and fix defects systematically |
| `refactor` | Improve code structure without changing behavior |
| `ui-ux-review` | Review UI quality, accessibility, and platform behavior |
| `code-review` | Final engineering quality gate |
| `test-review` | Assess test coverage quality and identify gaps |
| `skill-customization` | Adapt installed skills to the project's architecture, tooling, and conventions |
| `figma-to-flow` | Convert a Figma design into a static UI and navigation flow specification |

---

## Supported installation operations

### Install everything

```
Install all skills from <repository>.
```

### Install selected skills

```
Install the following skills from <repository>:
- feature-analysis
- feature-development
- code-review
```

### Update installed skills

```
Update the installed skills from <repository>.
```

---

## Step-by-step installation procedure for Claude

When asked to install or update skills, follow this procedure exactly.

### Step 1 — Read the manifest

Read `skills.json` in this repository to get the canonical list of skill names, versions, and paths.

### Step 2 — Inspect the target project

```bash
ls -la <target-project-root>
ls -la <target-project-root>/.claude/ 2>/dev/null || echo "No .claude directory"
ls -la <target-project-root>/.claude/skills/ 2>/dev/null || echo "No skills directory"
```

Identify:
- Whether `.claude/` exists.
- Whether `.claude/skills/` exists.
- Which skills are already installed.
- Whether any installed skills have been project-customized (content differs from this repository's version).

### Step 3 — Create skill directory if needed

```bash
mkdir -p <target-project-root>/.claude/skills
```

### Step 4 — For each skill to be installed

Apply the following decision logic for every skill:

#### Skill does not exist in target project
Install it:
```bash
cp -r skills/<skill-name> <target-project-root>/.claude/skills/<skill-name>
```
Record as: **installed (new)**.

#### Skill exists and matches this repository's version
Already up to date. Do nothing.
Record as: **already installed (up to date)**.

#### Skill exists and differs from this repository's version

1. Read the installed `SKILL.md`.
2. Read this repository's `SKILL.md`.
3. Determine whether the installed version has project-specific customizations. A skill is considered project-customized if it contains a `## Project-Specific Configuration` section (added by `skill-customization`) or if its universal content was manually modified.

**If no project-specific customizations**: Offer to replace with the repository version. Wait for explicit confirmation before overwriting.

**If a `## Project-Specific Configuration` section is present**: **Do not overwrite.** Report as "installed (project-customized — skipped)." Inform the user: to update this skill while preserving the project customization, run the `skill-customization` skill with the instruction to update after a universal skill update.

**If the universal content was manually modified (no project section separator)**: **Do not overwrite.** Report as "installed (manually customized — skipped)." The user must explicitly request replacement.

### Step 5 — Verify installation

After processing all skills:

```bash
find <target-project-root>/.claude/skills -name "SKILL.md" | sort
```

Confirm each requested skill's `SKILL.md` is present.

### Step 6 — Check for CLAUDE.md

```bash
ls <target-project-root>/CLAUDE.md 2>/dev/null || echo "No CLAUDE.md"
```

If `CLAUDE.md` does not exist:
> "No CLAUDE.md found. Run the `project-context` skill to analyze this project and generate a project-specific CLAUDE.md."

If it exists, no action needed.

### Step 7 — Report

```
INSTALLATION REPORT
===================

Skills installed (new):
- [list]

Skills already up to date:
- [list]

Skills skipped (project-customized):
- [list — explain that user must explicitly request replacement]

Skills not installed (error):
- [list — with reason]

CLAUDE.md: [present / not present — recommendation if absent]

Recommended next steps:
1. If no CLAUDE.md: Run the `project-context` skill to analyze the project and generate CLAUDE.md.
2. After CLAUDE.md exists: Run the `skill-customization` skill to adapt the installed skills
   to the project's architecture, tooling, and conventions.
```

---

## Safety rules

These rules are non-negotiable. Do not deviate from them.

**Never touch application source code** during installation or update.

**Never overwrite a project-customized skill** without explicit user confirmation. A customized skill is one whose content differs from this repository's version in ways that appear intentional (project-specific rules, commands, or examples).

**Never modify `CLAUDE.md`** as part of installation. That is the responsibility of the `project-context` skill.

**Never delete existing `.claude/` content** unrelated to the skills being installed.

**Installation is idempotent.** Running it twice on a project that already has the skills should result in no changes.

---

## Handling updates

When asked to update skills:

1. Follow the same procedure as installation.
2. For each skill that differs from the repository version:
   - If not project-customized: replace and record as "updated".
   - If project-customized: skip and report as "skipped — project-customized".
3. For each skill that is identical to the repository version: record as "already up to date".
4. Never downgrade — if the installed version appears newer than the repository version, report it and skip.

---

## Uninstalling skills

To remove a specific skill:
```bash
rm -rf <target-project-root>/.claude/skills/<skill-name>
```

To remove all skills:
```bash
for skill in project-context codebase-analysis feature-analysis feature-development \
             bug-fix refactor ui-ux-review code-review test-review skill-customization \
             figma-to-flow; do
  rm -rf <target-project-root>/.claude/skills/$skill
done
```

This does not affect `CLAUDE.md` or any other `.claude/` configuration.

---

## Adding project-specific skills

After installation, add custom project-specific skills alongside these ten:

1. Create `.claude/skills/<custom-skill-name>/SKILL.md` in the target project.
2. Write the skill using the same structure as the installed skills.

Do not add project-specific skills to this shared repository. Project-specific rules belong in `CLAUDE.md`.

---

## Using the skills.json manifest

The `skills.json` file at the root of this repository is machine-readable:

```json
{
  "version": "2.0.0",
  "skills": [
    {
      "name": "skill-name",
      "version": "1.0.0",
      "description": "...",
      "path": "skills/skill-name/SKILL.md"
    }
  ]
}
```

Use it to:
- Get the canonical list of skill names without reading each directory.
- Compare installed skill versions against repository versions.
- Determine which skills are new in a repository update.
