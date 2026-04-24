## quackle-mobile

> This repository builds a browser-local Scrabble application backed by Quackle compiled to WebAssembly.

# CLAUDE.md

## Project context
This repository builds a browser-local Scrabble application backed by Quackle compiled to WebAssembly.

The desired architecture is:
- TypeScript web UI
- dedicated Web Worker for solver runtime
- single-threaded WASM first
- offline-built lexicon assets
- parity testing between native harness and WASM build

## Rules
- Default to single-threaded WASM unless a task explicitly targets optional threaded mode.
- Keep the browser-facing engine API narrow and typed.
- Preserve a native characterization harness and compare WASM against it.
- Never introduce a server dependency for core solving behavior.
- Treat docs as part of the implementation, not an afterthought.
- Record milestone progress, commands, results, blockers, and decisions in `docs/status.md`.

## Required reading order
1. `docs/spec.md`
2. `docs/architecture.md`
3. `docs/testing-plan.md`
4. `docs/milestones.md`
5. `docs/legal.md`
6. `docs/status.md`

## Working style
- Favor incremental, testable changes.
- Surface assumptions explicitly.
- Prefer deterministic tests and fixture-based comparisons.
- Keep public interfaces small and stable.
- Build the simplest thing that satisfies the current milestone.

## Quality bar
Before finishing a task:
- run relevant verification commands
- fix failures when feasible
- update docs if the plan or commands changed
- update `docs/status.md` with:
  - date/time
  - milestone/task worked on
  - assumptions
  - files changed
  - commands run
  - results
  - open questions / next steps

## Special guidance
- For engine changes, think about native harness parity first.
- For worker changes, think about cancellation, staleness, and initialization sequencing.
- For asset changes, think about versioning, checksums, and corruption recovery.
- For UI changes, think mobile-first and touch-first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blackrobot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
