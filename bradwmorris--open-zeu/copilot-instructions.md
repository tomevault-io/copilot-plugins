## open-zeu

> This repo is a local multi-project task system for AI project managers.

# open-zeu

This repo is a local multi-project task system for AI project managers.

## Read Order

1. Read this file.
2. Read the target project's `AGENTS.md`.
3. Read that project's `data.json`.
4. Read the RD linked on the task.

## Repo Map

- `_skills/` = shared meta workflows
- `projects/*/AGENTS.md` = project instructions
- `projects/*/data.json` = backlog
- `projects/*/rds/` = requirements docs
- `projects/*/skills/` = reusable workflows
- `_templates/` = project skeleton
- `ui/` = local kanban

## Rules

- Do not start work without an RD.
- Keep tasks small and testable.
- Keep docs short.
- Do not commit secrets.
- If you want memory or MCP hooks, wire in your own system.

## Optional Memory Pattern

Use any store you want. Example:

- Before work: `mcp__notes__query`
- After work: `mcp__notes__create`

The pattern matters more than the tool.

## Workflow

### `/prep`

1. Clarify the task.
2. Tighten the RD.
3. Make the done criteria testable.

### `/run`

1. Read the task.
2. Read the RD.
3. Do the work.
4. Update the task status.

### `/wrap`

1. Summarize what changed.
2. Save the important decisions.
3. Clean up loose ends.

## Shared Skills

- `_skills/skill_prep.md`
- `_skills/skill_run.md`
- `_skills/skill_review.md`
- `_skills/skill_wrap.md`

## Demo Managers

This starter ships with `PM1`, `PM2`, `PM3`, and `PM4`.
Rename or delete them after cloning.

---
> Source: [bradwmorris/open-zeu](https://github.com/bradwmorris/open-zeu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
