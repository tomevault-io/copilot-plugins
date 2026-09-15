## nine-fives

> How this repo gets built. Follow it unless told otherwise.

# AGENTS.md — Nine-Fives

How this repo gets built. Follow it unless told otherwise.

## Process

- Plan before code. Discuss the shape first; write code only when asked.
- Boring wins: stdlib first, no new dependencies, fewest files, smallest diff.
- One recommendation, not a menu.

## Go conventions

- Idiomatic boring Go: short receivers, explicit error returns, `gofmt` clean.
- Behavior comments (intent, edge cases), never syntax comments.
- Table-driven tests for core logic; test behavior, not internals.
- Every non-empty sim `Result` carries a `Why` — the CLI and debrief reuse it.
- Checks before done: `go vet ./...`, `gofmt -l .`, `go test ./...`.

## Architecture

- `sim/` is headless: no rendering imports, no rendering logic.
  Render may import sim, never the reverse.
- `Component` = `Info() + Cost() + Handle(Tick)`. Keep the surface small.
- Placeholder costs are fine mid-build; wire them properly at the planned
  step and say so.
- Module: `github.com/codetesla51/nine-fives` (lowercase).

## Git

- Conventional atomic commits (`feat(sim): ...`, `docs: ...`).

---
> Source: [codetesla51/nine-fives](https://github.com/codetesla51/nine-fives) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
