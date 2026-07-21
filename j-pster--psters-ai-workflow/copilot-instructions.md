## psters-ai-workflow

> This file is the operational contract for `plugins/psters-ai-workflow`.

# Plugin Agent Contract

This file is the operational contract for `plugins/psters-ai-workflow`.

## Core principles

1. User control first
   - The user chooses workflow path and quality gates.
   - Do not silently auto-switch between `/pwf-work`, `/pwf-work-light`, `/pwf-work-tdd`, or `/pwf-work-plan`.
2. Predictable and repeatable execution
   - Prefer explicit steps and deterministic outputs.
   - Keep side-effect commands explicit and non-automatic.
3. Durable documentation
   - `docs/` is operational memory.
   - Keep docs aligned with implemented state.

## Agent naming convention (collision-safe)

When referencing agents in prompts, use fully-qualified names:

- `psters-ai-workflow:research:<agent-name>`
- `psters-ai-workflow:review:<agent-name>`
- `psters-ai-workflow:docs:<agent-name>`
- `psters-ai-workflow:workflow:<agent-name>`
- `psters-ai-workflow:design:<agent-name>`

Do not rely on short names alone in cross-plugin environments.

## Path conventions

- Avoid `@plugins/...` references in commands and skills.
- Use logical plugin paths for internal assets:
  - `rules/...`, `skills/...`, `agents/...`, `presets/...`
- Use project-owned paths for user repository data:
  - `docs/...`
- For plugin scripts, prefer `${CURSOR_PLUGIN_ROOT}` to avoid repo-coupled paths.

## Side-effect command policy

Commands that write files, deploy, or commit must be explicit user actions and should include:

- `disable-model-invocation: true` in command frontmatter (when supported by the runtime)

Typical side-effect commands:

- `pwf-work`, `pwf-work-plan`, `pwf-work-light`, `pwf-work-tdd`
- `pwf-plan`, `pwf-brainstorm`, `pwf-clarify`, `pwf-checklist`, `pwf-doc`
- `pwf-doc-foundation`, `pwf-doc-runbook`
- `pwf-doc-capture`, `pwf-doc-refresh`
- `pwf-setup`, `pwf-commit-changes`, `pwf-aws-lambda-deploy`

## Multi-agent orchestration

Parallel subagents are encouraged when tasks are independent.

Guardrails:

- Keep one orchestrator owner.
- Use explicit role boundaries.
- Merge outputs into a single deterministic decision.
- Ask user before broad/high-risk autonomous execution.

## Minimal validation before release/commit

Run:

1. `node scripts/validate-template.mjs`
2. lint/diagnostic checks for edited files
3. docs consistency check for updated workflow commands

For release-impacting plugin changes, ensure:

- `CHANGELOG.md` updated appropriately
- `.cursor-plugin/marketplace.json` metadata/version consistency

---
> Source: [J-Pster/Psters_AI_Workflow](https://github.com/J-Pster/Psters_AI_Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
