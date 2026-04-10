## tradingsystem

> These instructions apply to all work inside `NewTradingSystem`.

# AGENTS

## Scope

These instructions apply to all work inside `NewTradingSystem`.

## Core Rule

The code, the docs, and the migration status must tell the same story.

If code changes but the docs still describe the old behavior, the work is incomplete.

## Documentation Freshness Contract

When a change affects architecture, runtime behavior, strategy behavior, broker support, operator workflow, or current project status, update the matching documentation in the same change:

- `README.md` for entrypoints, commands, and top-level capabilities
- `docs/current-state.md` for what is real today versus not implemented yet
- `docs/legacy-inventory.md` for the concrete extraction and discard map
- `docs/legacy-extraction-portal.html` for the human-friendly published architecture and migration view
- `docs/documentation-freshness-contract.md` for the documentation process itself if the process changes
- `tasks/master-roadmap.md` for status and backlog movement
- any relevant plan under `docs/plans/`

Do not claim a feature is shipped if these documents still describe the previous state.

## Legacy Extraction Rules

Every imported asset from the legacy `trading` repo must be classified before implementation:

- `extract`: preserve the behavior with minimal translation
- `redesign`: keep the idea but rebuild it against the new platform boundaries
- `reference-only`: keep it as historical input, not runtime code
- `discard`: do not port it

Do not copy legacy workflows, data dumps, ad hoc scripts, or file-based state blindly.

## Runtime State

- Keep `.env*`, `var/`, generated dashboard data, and any local runtime state out of git.
- Keep paper-only as the default unless an explicit task says otherwise.
- Do not let strategies talk directly to broker APIs.

## Verification Rule

Before closing a task, verify the thing that changed:

- code changes: run tests and the relevant runtime command
- docs changes: check links, structure, and rendered HTML when applicable
- migration status changes: confirm they match the codebase truth

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jnrahme)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jnrahme)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
