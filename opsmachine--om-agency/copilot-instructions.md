## om-agency

> > **MANDATORY FIRST ACTION:** Before responding to ANY work request, read `~/.claude/skills/workflow-router/SKILL.md` and follow its Quick Start checklist. Announce `🎯 Workflow Manager active. Checking project state...`, orient, then propose the next skill. Do NOT skip this step.

# Skills System

> **MANDATORY FIRST ACTION:** Before responding to ANY work request, read `~/.claude/skills/workflow-router/SKILL.md` and follow its Quick Start checklist. Announce `🎯 Workflow Manager active. Checking project state...`, orient, then propose the next skill. Do NOT skip this step.

Modular skill ecosystem for Claude Code and Cursor. Skills are composable workflow nodes with typed contracts.

## What This Repo Is

This is the skill system itself — not a product codebase. Changes here affect how agents work across all projects.

## Key References

| Need | File |
|------|------|
| Workflow navigation & state model | `skills/workflow-router/SKILL.md` |
| System design history | `~/.claude/docs/OPERATIONAL_SYSTEM.md` |
| Skill contracts & shared primitives | `skills/shared/` |

## When Editing Skills

- Follow existing patterns in other SKILL.md files
- Every skill needs a `contract:` block in frontmatter
- Reference `shared/spec-io.md` and `shared/github-ops.md` instead of duplicating
- Test by invoking the skill in a real project

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/opsMachine) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
