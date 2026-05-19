## fdk-labs

> This is an Agent Skills Framework repository containing skill definitions for AI agents. Skills are Markdown files with YAML frontmatter that define AI agent behaviors for common development tasks.

# AGENTS.md

## Project Overview

This is an Agent Skills Framework repository containing skill definitions for AI agents. Skills are Markdown files with YAML frontmatter that define AI agent behaviors for common development tasks.

## Directory Structure

```
agent-skills/
  skills/
    {skill-name}/           # kebab-case directory name
      SKILL.md              # Required: skill definition
      scripts/              # Optional: executable scripts
        {script-name}.sh    # Bash scripts (preferred)
    {skill-name}.zip        # Required: packaged for distribution
```

## Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `vercel-deploy`, `log-monitor`)
- **SKILL.md**: Always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh` (e.g., `deploy.sh`, `fetch-logs.sh`)
- **Zip file**: Must match directory name exactly: `{skill-name}.zip`

## Available Skills

Invoke skills using `/skill-name`:

| Skill                      | Description                                        |
| -------------------------- | -------------------------------------------------- |
| `/commit-push`             | Stage, commit, and push to current branch          |
| `/commit-push-branch`      | Pull main, create branch, commit, and push         |
| `/create-pr`               | Generate PR description (arg: `fixes "#123"`)      |
| `/create-branch-and-pr`    | Branch, commit, push, and create pull request      |
| `/security-check-frontend` | Run npm audit and check Dependabot alerts          |
| `/improve-prompt`          | Analyze and improve a prompt (arg: prompt text)    |
| `/create-feature-issue`    | Create GitHub feature issue (Norwegian)            |
| `/create-bug-issue`        | Create GitHub bug issue (Norwegian)                |
| `/create-ux-issue`         | Create GitHub UX issue (Norwegian)                 |
| `/tree-shaking-js`         | Check for unused JS dependencies and plan removals |

## Skill File Format

Skills use YAML frontmatter with these fields:

```yaml
---
description: Short description shown in skill list
argument-hint: [optional placeholder for arguments]
---
```

Use `$ARGUMENTS` in the skill body to reference user-provided arguments.

## Important Rules

- Never run `yarn skills:add --yes` without asking for permission first. It is not recommended to use this command for verification.

---
> Source: [fellesdatakatalog/fdk-labs](https://github.com/fellesdatakatalog/fdk-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
