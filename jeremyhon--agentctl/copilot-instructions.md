## agentctl

> Instructions for coding agents working in `agent-orchestrator`.

# AGENTS.md

Instructions for coding agents working in `agent-orchestrator`.

## Scope

- This file applies to the entire repository.
- Follow it together with `docs/v1-decisions.md` and `docs/requirements-and-plan.md`.

## Product Direction (Do Not Drift)

- Local-first, single-process runtime.
- SQLite is the canonical datastore in v1.
- Minimal local setup: no required Docker/services for normal development.
- Fast dogfooding is prioritized over broad feature scope.
- Architecture is OpenClaw-inspired but intentionally simpler.

## Current Build Priority

1. Phase 3: remote/mobile reliability (auth, cursor replay, idempotency).
2. Improve Phase 1/2 robustness (idempotency keys, better stderr/log shaping, run ownership checks).
3. Add streaming transport (WS/SSE) on top of cursor replay API.

Do not jump ahead unless explicitly requested.

## Runtime Components

- `agentd`: daemon/API/runtime owner.
- `agentctl`: thin CLI client to `agentd`.
- `orchestrator/db.py`: SQLite schema and storage access.
- `orchestrator/state_machine.py`: authoritative run-state transitions.
- `skills/agent-orchestrator-cli/SKILL.md`: canonical operator skill for running this system.

## Non-Negotiables

- Preserve run state machine semantics and terminal immutability.
- Keep API and CLI outputs machine-readable JSON.
- Persist important run/session transitions as events.
- Keep changes dependency-light unless a new dependency is clearly justified.
- Prefer small, composable modules and incremental commits.

## Development Commands

- Start daemon: `./bin/agentd`
- Use CLI: `./bin/agentctl ...`
- Run tests: `make test`

## Change Checklist

- Add/adjust tests for behavior changes.
- Update docs when decisions, contracts, or commands change.
- Keep examples in `README.md` accurate.
- If `agentctl` commands/flags/output change, update `skills/agent-orchestrator-cli/SKILL.md` in the same PR.
- After every repo change, restart any running `agentd` process so the live daemon matches current `main`.
- Avoid unrelated refactors while implementing a phase task.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeremyhon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
