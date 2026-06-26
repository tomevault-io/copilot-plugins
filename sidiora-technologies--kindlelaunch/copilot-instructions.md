## kindlelaunch

> This repository is an AI-agent workspace. The following rules apply to any

# AGENTS.md — AI Agent Guide

This repository is an AI-agent workspace. The following rules apply to any
automated or semi-automated assistant editing this codebase.

## Ground truth

Always read the authoritative sources before architecting or deciding:

1. `README.md` — high-level architecture and commands.
2. `knowledge/kindlelaunch.frozen.kvx` — frozen master plan for the backend
   rewrite.
3. `CLAUDE.md` — this file's sibling, contains hard project rules.
4. `contracts/deployments/paxeer-addresses.json` — canonical on-chain addresses.
5. `Makefile` and `.github/workflows/ci.yml` — how the project is built and
   validated.

## Build and test discipline

- Run `make ci` before considering any Go change complete. It runs `build`, `vet`,
  `fmt-check`, `lint`, `race`, and `cover-check`.
- For contract changes, run `pnpm compile`, `pnpm test`, and `pnpm lint:sol` in
  `contracts/`.
- **Never fake test results.** Tests must exercise real code paths and real types.
  Do not write stub mocks or placeholders that make tests pass.
- **Coverage is mandatory.** If your change drops a module below its gate, add
  tests until it is green again.
- **No float for money.** All token, price, volume, and PnL arithmetic uses
  `math/big.Int` or `uint256`.

## Change discipline

- **Fix root causes, not symptoms.** One-line upstream fixes are preferred over
  large downstream workarounds.
- **Minimal changes.** Do not refactor unrelated code while fixing a bug.
- **No commits from the agent.** The user drives commits via their own pre-commit
  hook. Do not run `git commit` or `git push`.
- **Do not modify production infrastructure** (validators, genesis, rate cards,
  deployed contract parameters) without explicit user approval.
- **Add regression tests** for every bug fix.

## What not to do

- Do not add or delete comments or documentation unless asked, or unless the
  change genuinely requires it.
- Do not create random files that clutter the workspace. Every new file must be
  justified by the task.
- Do not use emojis, purple gradients, or glow effects in UI or docs.
- Do not use border strokes for visual separation in UI; use background-color
  contrast instead.
- Do not chain prod-affecting autonomous actions (deploys, machine updates,
  background builds) without informing the user and getting their go-ahead.

## Escalation

If a task touches any of the following, stop and ask the user for explicit
confirmation before proceeding:

- Smart contract upgrade paths, `Timelock` parameters, or governance actions.
- Production RPC endpoints, private keys, or API credentials.
- The validator cluster at `147.93.139.18` (host id `validator-cluster`).
- Genesis state, private validator keys, or chain snapshots.
- Rate cards or PAX/USD recalibration in the gateway.

## Persistence

This project uses the Matrix cortex for persistent memory. When you learn
something durable about the project, the user's preferences, or the environment,
record it with `cortex_remember_*`. At the start of every session, run
`cortex_recall()` to load hard rules and prior context.

---
> Source: [Sidiora-Technologies/KindleLaunch](https://github.com/Sidiora-Technologies/KindleLaunch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
