---
name: setup-claude-code-setting
description: Bootstrap Claude Code project settings by importing skills from a reference project. Copies manage-skills and verify-implementation as-is, then analyzes and adapts verify-* skills to match the current repository's architecture. Use when setting up Claude Code skills for a new project from an existing reference, or when the user invokes /setup-claude-code-setting.
argument-hint: "<source-directory> [optional: specific skill names to include]"
---

# Setup Claude Code Settings

## Purpose

Bootstraps a project's `.claude/skills/` directory by importing and adapting skills from a reference project:

1. **Copy foundation skills** — `manage-skills` and `verify-implementation` are copied and adjusted for the current project
2. **Discover verify skills** — Scan the source directory for all `verify-*` skills
3. **Analyze current project** — Understand the architecture, patterns, and conventions of the target repository
4. **Adapt skills** — Rewrite or modify each verify skill to match the current project's structure
5. **Generate CLAUDE.md** — Create a project-level CLAUDE.md with architecture overview and skills table

## Workflow

### Step 1: Validate Input

Read the source directory argument. Verify it contains `.claude/skills/`:

```bash
ls <source-directory>/.claude/skills/
```

If not found, report error and exit.

Also check if the current project already has `.claude/skills/`. If so, inform the user and use `AskUserQuestion` to confirm whether to overwrite, merge, or abort.

### Step 2: Discover Source Skills

List all skills in the source directory:

```bash
ls <source-directory>/.claude/skills/
```

Categorize them:
- **Foundation skills** (always copied): `manage-skills`, `verify-implementation`
- **Verify skills** (candidates for adaptation): all `verify-*` directories
- **Other skills**: any remaining skills (report to user, skip unless requested)

Display to user:

```markdown
## Source Skills Found

| Category | Skills |
|----------|--------|
| Foundation (will copy) | manage-skills, verify-implementation |
| Verify (will adapt) | verify-code-convention, verify-model-registration, verify-test-coverage, ... |
| Other (skipped) | pr-reviewer, ... |
```

### Step 3: Analyze Current Project Architecture

Explore the current repository to understand its structure. Gather:

1. **Project layout** — Top-level directories, source code location, test structure
2. **Language & framework** — Primary language, frameworks, build tools
3. **Dependency management** — uv, poetry, npm, cargo, etc.
4. **Linting & formatting** — ruff, eslint, prettier, etc. and their config files
5. **Model/registration patterns** — How components are registered (factory, enum, config, etc.)
6. **Test patterns** — Test framework, directory structure, naming conventions
7. **Entry points** — Main scripts, CLI tools, API servers
8. **Constants & configuration** — Enums, config files, environment variables

Use these tools to gather information:

```bash
# Project structure
ls -la
find . -name "*.py" -o -name "*.ts" -o -name "*.rs" -o -name "*.go" | head -50

# Config files
cat pyproject.toml 2>/dev/null
cat package.json 2>/dev/null
cat Cargo.toml 2>/dev/null

# Test structure
ls tests/ 2>/dev/null
ls test/ 2>/dev/null

# Linting config
cat ruff.toml 2>/dev/null
cat .eslintrc* 2>/dev/null
```

### Step 4: Read Source Skills

For each skill to import, read the full SKILL.md:

```bash
cat <source-directory>/.claude/skills/<skill-name>/SKILL.md
```

For verify skills, extract the key elements to understand what each skill checks:
- **Purpose** — What categories of verification
- **Related Files** — Source-project-specific file paths
- **Workflow** — Detection commands and patterns
- **Exceptions** — What is not a violation

### Step 5: Adapt Foundation Skills

#### manage-skills

Copy and modify:
1. Update the **Registered Verification Skills** table to list the skills being created for this project
2. Update **Covered File Patterns** to match this project's directory structure
3. Update the **Related Files** section
4. Update **Exceptions** if the project uses different lock files or config formats

#### verify-implementation

Copy and modify:
1. Update the **Skills to Execute** table to list only the verify skills being created for this project
2. No other changes needed — the workflow is generic

### Step 6: Adapt Verify Skills

For each verify skill, apply this decision process:

```
IF the skill's domain exists in the current project:
    IF the patterns are similar (e.g., both use ruff, both have model registration):
        → MODIFY: Update file paths, commands, and tool references
    ELSE IF the domain exists but patterns differ significantly:
        → REWRITE: Keep the skill structure but rewrite checks for this project's patterns
ELSE:
    → SKIP: The skill's domain doesn't apply to this project
```

**When MODIFYING a skill:**
- Replace source project file paths with current project paths
- Replace dependency manager commands (e.g., `poetry run` → `uv run`)
- Update grep/glob patterns to match current file structure
- Update "Current registered models/components" lists
- Update Related Files table with actual paths (verify with `ls`)

**When REWRITING a skill:**
- Keep the same SKILL.md structure (frontmatter, Purpose, When to Run, Related Files, Workflow, Output Format, Exceptions)
- Write new checks based on the current project's architecture discovered in Step 3
- Each check must have: Tool, command/pattern, PASS/FAIL criteria, Fix instructions
- Verify all file paths exist

**When SKIPPING a skill:**
- Report to user why it was skipped (domain not applicable)

Present the adaptation plan to user with `AskUserQuestion` before proceeding:

```markdown
### Skill Adaptation Plan

| Source Skill | Action | Reason |
|-------------|--------|--------|
| verify-code-convention | MODIFY | Both projects use ruff + PEP 8 |
| verify-model-registration | REWRITE | Different registration pattern (Factory → Enum + dict) |
| verify-test-coverage | MODIFY | Similar test structure |
| verify-config-consistency | SKIP | No Hydra configs in this project |
```

### Step 7: Create Skills in Current Project

Create the `.claude/skills/` directory structure:

```bash
mkdir -p .claude/skills/<skill-name>
```

Write each adapted SKILL.md file.

### Step 8: Generate CLAUDE.md

If the project does not have a `CLAUDE.md` at the root, create one with:

- **Project Overview** — Brief description based on README or project structure
- **Common Commands** — Build, test, lint commands discovered in Step 3
- **Architecture** — Key modules, entry points, patterns
- **Code Style** — Language version, linter config
- **Skills** — Table of all created skills with descriptions

If `CLAUDE.md` already exists, append or update only the **Skills** section.

### Step 9: Verification

After all files are created:

1. Verify all skill directories exist:
   ```bash
   ls .claude/skills/
   ```

2. For each skill, verify Related Files paths exist:
   ```bash
   # For each path in Related Files tables
   ls <file-path> 2>/dev/null || echo "MISSING: <file-path>"
   ```

3. Verify the **Registered Verification Skills** table in manage-skills matches the **Skills to Execute** table in verify-implementation

4. Verify the **Skills** table in CLAUDE.md lists all created skills

### Step 10: Summary Report

```markdown
## Setup Complete

### Skills Created: N

| Skill | Action | Source |
|-------|--------|--------|
| manage-skills | Copied & adjusted | <source-dir> |
| verify-implementation | Copied & adjusted | <source-dir> |
| verify-code-convention | Modified | <source-dir> |
| verify-model-registration | Rewritten | <source-dir> |
| verify-test-coverage | Modified | <source-dir> |

### Skills Skipped: M
- verify-config-consistency — No Hydra configs in this project

### Files Created:
- `.claude/skills/*/SKILL.md` (N files)
- `CLAUDE.md` (created / updated)

### Next Steps:
- Run `/verify-implementation` to test all skills
- Run `/manage-skills` to check for gaps
```
