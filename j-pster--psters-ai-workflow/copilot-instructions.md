## docs-scope-boundary

> Keep plugin docs scope separate from project docs scope.


# Docs Scope Boundary

Never mix these two `docs/` scopes:

1. **Plugin docs scope (this repository):**
   - Root `docs/` in this workspace documents the plugin itself.
2. **Project docs scope (target project):**
   - `docs/` created/updated by workflow commands in the user's project.

## Required behavior

- For `/pwf-setup`, `/pwf-work`, `/pwf-work-plan`, `/pwf-work-light`, `/pwf-work-tdd`, always operate on the **target project** docs tree.
- Do not treat this repository root `docs/` as the target project docs unless the user explicitly says this repo is the target project.
- If scope is ambiguous, ask which docs scope to use before writing files.

## Canonical project docs paths

Use these paths (no `pwf-` directory prefixes):

- `docs/infrastructure.md`
- `docs/architecture.md`
- `docs/integrations.md`
- `docs/environments.md`
- `docs/glossary.md`
- `docs/runbooks/`
- `docs/runbooks/README.md`
- `docs/brainstorms/`
- `docs/plans/`
- `docs/work-plans/`
- `docs/workflow/operational-overrides.md`
- `docs/solutions/`
- `docs/modules/`
- `docs/features/`
- `docs/lambdas/`
- `docs/decisions/`

---
> Source: [J-Pster/Psters_AI_Workflow](https://github.com/J-Pster/Psters_AI_Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
