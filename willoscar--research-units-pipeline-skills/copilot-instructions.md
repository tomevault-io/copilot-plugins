## research-units-pipeline-skills

> This is the minimal repo-level instruction file for Codex and other coding agents working in this repository.

# AGENTS.md

This is the minimal repo-level instruction file for Codex and other coding agents working in this repository.

## Scope

- Treat this file as the top-level repo-global note, not as the full execution contract.
- Use it for repository-wide rules only.
- Read pipeline files, skill files, and standards documents for workflow-specific behavior.

## Why this file still exists

- Local tooling currently uses `AGENTS.md` as the repo-root marker.
- Do not delete or rename it until repo-root discovery is migrated away from `AGENTS.md`.

## Canonical contracts live elsewhere

- Workflow execution contracts: `pipelines/*.pipeline.md`
- UNITS and checkpoint conventions: `templates/UNITS.*.csv`, `templates/units.schema.md`, `SKILLS_STANDARD.md`
- Skill-specific behavior: `.codex/skills/<skill>/SKILL.md`

## Repo-global rules agents should follow

- Keep generated run artifacts under `workspaces/<name>/`; do not write workspace outputs into the repo root.
- Prefer repo skills under `.codex/skills/`; if a capability is missing, add or refactor a skill instead of doing the work ad hoc.
- For executable pipelines, follow the selected pipeline's artifact contract, checkpoint contract, and approval flow rather than duplicating those rules here.

---
> Source: [WILLOSCAR/research-units-pipeline-skills](https://github.com/WILLOSCAR/research-units-pipeline-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
