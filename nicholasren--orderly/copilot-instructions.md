## orderly

> Run `invoke -l` to see everything this project can do. That list is the project's

# orderly — agent onboarding

Run `invoke -l` to see everything this project can do. That list is the project's
capability surface; prefer those tasks over ad-hoc commands (raw `curl`, hand-rolled
scripts). If a task exists for it, use the task.

## Environment

`uv sync` once, then either `uv run invoke <task>` or activate `.venv` (direnv users:
`direnv allow` and forget about it).

## Know before you code

- [docs/architecture.md](docs/architecture.md) — layers, where code goes, error conventions
- [docs/testing.md](docs/testing.md) — how tests run, house style, what to test where
- [docs/deployment.md](docs/deployment.md) — environments, how changes reach them

## Hard rules (enforced, not advisory)

- `invoke lint` must pass: ruff + import-linter. The layering contract
  (`api → service → repository → domain`) is checked on every run — a change that
  imports against the layers will fail CI, so fix the design, not the contract.
- `invoke test` must pass before any PR.
- CI runs exactly `invoke lint` and `invoke test`. If CI is red, reproduce with the
  same command locally — same code, same semantics.

## Team workflow

Work on a branch. Before opening a PR: `invoke lint && invoke test`. Then
`invoke pr` (pushes and opens it), `invoke pr-status` to watch CI, and
`invoke pr-comments` to read review feedback. If a CI check fails, run the same
invoke task locally — CI runs nothing you can't run on a laptop. Address review
comments with follow-up commits on the same branch; reply via
`gh pr comment --body "..."`.

## Quality gates

The path to production is gated, identically for humans and agents: `invoke lint`
(style + architecture contracts), `invoke test`, `invoke audit` (dependency CVEs) —
all enforced in CI — then a staging deploy with `invoke smoke-test` via the release
pipeline (`.github/workflows/release.yml`, triggered by a `v*` tag). Promotion to
prod is always an explicit human decision. Routine releases: use the `deploy` skill
in `.agents/skills/deploy/`.

## Conventions in one paragraph

Business rules live in the service layer; route handlers stay thin (translate HTTP ↔
domain, nothing else); the repository stays dumb (storage, no rules). Domain models
import nothing from other layers. New endpoints follow the pattern in
`src/orderly/api/routes.py`; new tests follow the style in `tests/` (plain functions,
one behavior per test, fixtures for setup).

---
> Source: [nicholasren/orderly](https://github.com/nicholasren/orderly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
