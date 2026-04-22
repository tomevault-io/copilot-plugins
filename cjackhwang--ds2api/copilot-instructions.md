## ds2api

> These rules apply to all agent-made changes in this repository.

# AGENTS.md

These rules apply to all agent-made changes in this repository.

## PR Gate

- Before opening or updating a PR, run the same local gates as `.github/workflows/quality-gates.yml`.
- Required commands:
  - `./scripts/lint.sh`
  - `./tests/scripts/check-refactor-line-gate.sh`
  - `./tests/scripts/run-unit-all.sh`
  - `npm run build --prefix webui`

## Go Lint Rules

- Run `gofmt -w` on every changed Go file before commit or push.
- Do not ignore error returns from I/O-style cleanup calls such as `Close`, `Flush`, `Sync`, or similar methods.
- If a cleanup error cannot be returned, log it explicitly.

## Change Scope

- Keep changes additive and tightly scoped to the requested feature or bugfix.
- Do not mix unrelated refactors into feature PRs unless they are required to make the change pass gates.

---
> Source: [CJackHwang/ds2api](https://github.com/CJackHwang/ds2api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
