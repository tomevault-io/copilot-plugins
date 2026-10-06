## goarrows

> Go Arrows is a small terminal puzzle written in Go. The playing field is covered

# Go Arrows — Project Rules

## Overview

Go Arrows is a small terminal puzzle written in Go. The playing field is covered
with straight and curved arrows. The player moves a cursor, selects an arrow head,
and fires: the arrow slides along its flight path. If the beam reaches the edge of
the field unobstructed, the whole arrow is removed; if something blocks it, a life
is lost. Clear every arrow to win the level.

## Tech stack

- Go `1.26.1`, module path `goarrows`.
- Only external dependency: `github.com/gdamore/tcell/v2` (terminal UI). Avoid adding
  new dependencies without a strong reason.
- RNG uses the standard library `math/rand/v2` (PCG generators). Do not use the
  global `math/rand` source.
- Requires a UTF-8 terminal at runtime (box-drawing and arrow glyphs).

## Architecture and package boundaries

Keep game logic independent of the terminal. The presentation layer depends on the
logic, never the reverse.

- `game` — pure logic, no `tcell`/terminal imports. Owns `Board`, `Cell`,
  `Direction`, the port/connection bitmask, level parsing (`ParseLevel`,
  `ParseLevelString`), validation (`ValidateBoard`, `ValidatePartialBoard`),
  firing (`TryFire`, `RayEscapes`, `PathFromHead`), procedural generation
  (`GenerateBoard`/`GenGrow`, `growPlayfulEnough`, `VerifyGreedyFirstClearsBoard`),
  the backtracking `VerifySolvable` (tests only), and on-demand level generation
  (`NewLevels(seed)` plus `Levels.At`, which generate boards lazily and cache them
  in a memo, deterministic per seed).
- `ui` — renders a `game.Board` to a `tcell.Screen`. Maps logical cell `(x, y)` to
  screen column `2*x` (row `y`); `GridSize` is `(2*w-1, h)`. Draws HUD, help/modal/
  generating overlays, and both builds (`BuildFireFrames`) and draws fire animation
  frames.
- `main` — owns only the tcell setup, input loop, HUD layout, status/overlay state,
  and fire animation timing/stepping (it requests frames from `ui.BuildFireFrames`).
  It must not contain game rules; route logic through `game`.

## File layout

Each file has one responsibility; add new code to the matching file rather than
growing a catch-all.

- `game`: `board.go` (`Board`, `Cell`, `Direction`, `Delta`), `game.go` (`Game`,
  `TryFire`, `RayEscapes`, `Won`/`Lost`), `ports.go` (port bitmask,
  `EffectivePorts`, `linked`, `directionFromTo`), `validate.go` (`ValidateBoard`,
  `ValidatePartialBoard`), `path.go` (`PathFromHead`), `level.go` (parsing),
  `gen.go` (generation policy and `GenerateBoard`), `gen_grow.go` (the grow
  algorithm), `paint.go` (polyline rasterization), `levels.go` (the on-demand
  `Levels` generator: `NewLevels`, `At`, `Count`, `SideLen`, `Ready`, `levelRNG`),
  `solvable.go` (`VerifyGreedyFirstClearsBoard`, `VerifySolvable`).
  `countInitialRayEscapes` is a test-only helper in `gen_test.go`.
- `ui`: `grid.go` (`DrawGrid`, `GridSize`), `overlay.go` (`DrawStr`, `FormatLives`,
  help/modal/generating overlays), `animation.go` (fire-animation frame construction:
  `BuildFireFrames` plus `buildPointerFrames`, `fireTravelCells`).
- `main`: `main.go` (event/render loop), `flags.go` (CLI flags, `resolveProceduralSeed`),
  `levels.go` (level/game glue: `loadLevels`, `newGameForLevel`, `resetLevel`),
  `cursor.go`, `fire.go` (fire outcome to status/modal), `animation.go`
  (fire-animation timing/stepping: `animState`, `tryStartFireAnimation`).

## Conventions

- Coordinates are row-major: iterate `y` then `x`, index flat slices as `y*W + x`.
  Always bound-check with `Board.InBounds` before `At`/`Set`.
- `Cell.R == 0` means empty. Wires are `─│┌┐└┘`. Heads are normalized to
  `▲▼◀▶`; ASCII `^ v V < >` are accepted on input and normalized via
  `normalizeHeadRune`.
- Use the `Direction` enum (`North/East/South/West`) with `Delta(d)` for movement;
  never hardcode `dx/dy` literals. Use `oppositeDir` rather than re-deriving.
- Thread an explicit `*rand.Rand` through generation functions for determinism;
  derive per-level seeds via `levelRNG(seed, idx)`. Do not call package-level rand.
- Library code in `game`/`levels` returns `error` for invalid boards or sizes
  rather than panicking. Panics are only acceptable in `main` for truly
  unrecoverable setup (e.g. a cached level failing to reload).
- Keep functions small and prefer pure helpers. Comments should explain intent and
  invariants (degree rules, mutual links, one head per component), not narrate code.

## Board invariants (preserve these)

- A head has exactly one body link (one effective port).
- A wire cell has degree 1 (tail) or 2 (internal).
- Each connected component contains exactly one head.
- Adjacency requires mutual consent (both cells expose the shared edge); two heads
  never link.
- Generated boards must pass `ValidatePartialBoard`, `growPlayfulEnough` (at most
  half the heads have a clear shot at start), and `VerifyGreedyFirstClearsBoard`
  (greedy row-major firing clears the board).

## Build and run

- Run: `go run .` (UTF-8 terminal required).
- Build: `go build` or `make`.
- Flags: `-lives N` (default `3`, `-1` for unlimited); `-seed N` (omit for a random
  clock-based base seed; pass to reproduce a level sequence).

## Testing

- Run the suite with `gotestsum --format dots -- -timeout 10s ./...` (via `make test`)
  or `go test -timeout 10s ./...`. Always keep the `-timeout 10s` bound so the suite
  stays bounded.
- Coverage: `make cover`.
- Build board fixtures inline with `ParseLevelString`; for intentionally invalid
  layouts use a no-validate helper (see `boardFromLinesNoValidate` in tests).
- Prefer table-driven tests. Tests depend on deterministic seeds:
  `resolveProceduralSeed` returns `0` under `testing.Testing()`.

---
> Source: [sergev/goarrows](https://github.com/sergev/goarrows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
