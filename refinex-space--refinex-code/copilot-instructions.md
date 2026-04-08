## refinex-code

> <!-- HARNESS:MANAGED FILE -->

<!-- HARNESS:MANAGED FILE -->
# Refinex-Code

This repository uses an agent-first control plane with concise routing,
resumable plans, and observable runtime surfaces.

## Start Here

1. Read this file first
2. Then read [Plans](docs/PLANS.md)
3. Route into the focused docs by task type
4. Read the closest local `AGENTS.md` before editing code

## Task Routing

| Task | Read first | Then |
| --- | --- | --- |
| Architecture / boundaries | `ARCHITECTURE.md` | `docs/DESIGN.md` |
| Planning / progress | `docs/PLANS.md` | `docs/exec-plans/active/` |
| Security / permissions | `docs/SECURITY.md` | related code and constraints |
| Reliability / rollback | `docs/RELIABILITY.md` | runtime docs and implementation |
| Observability / runtime debugging | `docs/OBSERVABILITY.md` | `docs/generated/` |
| Quality / evaluation | `docs/QUALITY_SCORE.md` | `docs/generated/` |
| Frontend / UX / product behavior | `docs/FRONTEND.md` | `docs/PRODUCT_SENSE.md` |
| Design evolution / multi-module coordination | `docs/DESIGN.md` | `docs/design-docs/index.md` |

## Ground Rules

- `AGENTS.md` is a routing map, not an encyclopedia
- Keep the root route short, current, and mechanically checkable
- Use progressive disclosure: root map -> focused docs -> local `AGENTS.md` -> source code
- Active work belongs in `docs/exec-plans/active/`
- Completed work belongs in `docs/exec-plans/completed/`
- Structural debt belongs in `docs/exec-plans/tech-debt-tracker.md`
- Structured facts and indexes must remain checkable and current

## Local AGENTS Coverage

- No additional local AGENTS yet

## Mechanical Entry Points

- Repo check: `python3 scripts/check_harness.py`
- Harness manifest: `docs/generated/harness-manifest.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Refinex-Space)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Refinex-Space)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
