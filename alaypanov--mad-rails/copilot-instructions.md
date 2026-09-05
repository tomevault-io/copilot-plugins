## mad-rails

> Rail Builder — a 2D top-view, touch-first rail-laying browser game. Core mechanic: rails are pre-placed on the board as **sliding tiles**; drag one and it slides **axis-locked along its row/column** through as many empty cells as the drag allows, stopping at the first barrier (2D, 15-puzzle style) to assemble a connected path between the start and end stations. The board is **packed**: off-corridor land is filled with movable **blank filler tiles**, leaving only ~2–3 empty holes as maneuvering room — so assembling the path means shuffling both rail tiles and blanks around the immovable anchors. Immovable anchors (tunnels through terrain) and denser terrain form maze-like corridors that constrain movement; tiles can't pass each other, so reaching each rail's slot requires shuffling. TypeScript + HTML5 Canvas + Vite. No external rendering/physics libraries; all drawing is procedural on a 2D canvas context.

## Project

Rail Builder — a 2D top-view, touch-first rail-laying browser game. Core mechanic: rails are pre-placed on the board as **sliding tiles**; drag one and it slides **axis-locked along its row/column** through as many empty cells as the drag allows, stopping at the first barrier (2D, 15-puzzle style) to assemble a connected path between the start and end stations. The board is **packed**: off-corridor land is filled with movable **blank filler tiles**, leaving only ~2–3 empty holes as maneuvering room — so assembling the path means shuffling both rail tiles and blanks around the immovable anchors. Immovable anchors (tunnels through terrain) and denser terrain form maze-like corridors that constrain movement; tiles can't pass each other, so reaching each rail's slot requires shuffling. TypeScript + HTML5 Canvas + Vite. No external rendering/physics libraries; all drawing is procedural on a 2D canvas context.

## Commands

```bash
npm run dev       # Vite dev server (http://localhost:5171, see vite.config.ts), HMR
npm run build     # tsc -b (typecheck) then vite build → dist/
npm run preview   # serve the production build
npx tsc --noEmit  # typecheck only
```

### Logic smoke test

There is no test runner. The pure-logic self-test lives at `test/smoke.ts` and is run by bundling with esbuild (shipped with Vite) and executing under node:

```bash
npx esbuild test/smoke.ts --bundle --platform=node --format=esm --outfile=test/smoke.mjs && node test/smoke.mjs
```

It generates levels with scramble disabled and asserts the train reaches the end on the solved board (200 levels) and that an empty board never reaches the end; it also unit-tests `canStep`/`stepRail` edge cases (barriers, immovables, occupied neighbors, reversibility), the multi-cell `slideDistance`/`slideRail` primitives (barrier stop, atomicity, reversibility), and `Blank` tiles (no ports, movable, act as barriers, slide reversibly), plus a generation check that the board is packed (blanks placed, ~2–3 holes left) and the `Empty` region is 2-connected. It proves scrambled levels are solvable by **asserting Wilson's theorem preconditions** (2-connected `Empty` region, not a cycle, ≥2 identical blanks, ≥1 hole, rail multiset preserved by the swap) on a sample of scrambled levels — the preconditions ARE the proof, since a complete solver is infeasible at these sizes (it can never prove a board unsolvable). The `test/smoke.mjs` bundle is generated artefact — delete it after running.

## Architecture

The code is split into **framework-agnostic game logic** (`src/game/`, no DOM/canvas) and **presentation/glue** (`src/render/`, `src/input.ts`, `src/layout.ts`, `src/loop.ts`, `src/main.ts`). Keep that boundary: all gameplay rules live in `src/game/` and should be unit-testable; `src/render/` only reads game state and draws.

### Rail connection model (the core)

Each rail piece exposes a set of **ports** (subset of N/E/S/W). Two adjacent cells connect iff both expose a port facing each other (mutual). `src/game/pieces.ts` defines port sets per `(PieceType, orientation)`; `Straight`/`Tunnel` have 2 orientations, `Curve` has 4, `Blank` has 1 (an **empty** port set — no ports). `Straight` (movable) sits on `Empty`; `Tunnel` (immovable, straight-only) sits on `Terrain`; `Curve` (movable) sits on `Empty` only; `Blank` (movable, no ports) sits on `Empty` — a filler tile that is not track (entering one derails the train in `resolvePath`).

`src/game/slide.ts` is the canonical barrier logic for the movement: a cell is *passable* iff in-bounds, `Empty`, and no rail. `canStep(level,r,c,dir)` returns whether the movable tile at `(r,c)` can step **one cell** in direction `dir` (N/E/S/W) — the target must be in-bounds passable (empty land, no rail), so tiles are blocked by terrain/other tiles (rails or blanks)/stations/border and can't pass or swap with each other. The logic is piece-agnostic: any non-immovable `cell.rail` (Straight/Curve/**Blank**) moves; a Blank is a movable barrier to others. `stepRail(level,r,c,dir)` performs the one-cell step (relocate + free source). `slideDistance(level,r,c,dir)` counts how far the tile can slide straight in `dir` before hitting a barrier, and `slideRail(level,r,c,dir,steps)` performs an **axis-locked multi-cell slide** of `steps` cells — atomic (no mutation unless the full slide fits within `slideDistance`), and reversible. `isImmovable` (Tunnel) tiles never move and act as barriers.

`src/game/pathfind.ts` `resolvePath()` walks this connection graph from the start station's single exit port, entering each rail from one side and leaving via its other port. It returns either a full path to the end station (win) or the dead-end cell where the connection breaks (crash location). Cycles count as a crash. **When changing piece definitions, station port semantics, or slide/barrier logic, `resolvePath` and the smoke test are the integration points to re-verify.**

Stations (`StationStart`/`StationEnd`) are single-port cells; `levelGen` points each station's `exit` along the carved corridor, and `resolvePath` checks the end station's port faces the incoming track.

### State machine

`src/game/game.ts` `Game` holds the single source of truth and transitions through `GameState`: `Building → Running → LevelComplete → next level`, or `Running → Crash → Building`. The game is **untimed**: the player slides rails in `Building`; after every committed slide `resolvePath` is re-run and a complete path **auto-launches** the train. A manual `sendTrain()` (SEND button) launches immediately to test an incomplete track — a crash returns to `Building` (no GameOver). `restart()` resets to level 1.

### Level generation & solvability

`src/game/levelGen.ts` `generateLevel(n, scramble = true)` scatters terrain (`GRID.terrainChance` forms maze-like corridors that force routing), carves a **guaranteed BFS corridor** between the two stations (the corridor may cross terrain since BFS treats all cells as passable), then **lays the solved rails** along it: straight-on-terrain → `Tunnel` (an immovable anchor), else movable `Straight`/`Curve`. A turn forced onto terrain is converted to `Empty` + `Curve` (a straight-only immovable can't turn). It then **enforces 2-connectivity**: `ensureBiconnected` (`src/game/connectivity.ts`, Tarjan articulation-point + bare-terrain repair) merges disconnected `Empty` pockets and clears 1-wide necks by flipping bare `Terrain` to `Empty` (never touching rails or `Tunnel`s); if it can't, the board is regenerated with lower terrain density (last resort: an open board). This is the precondition for the solvability theorem below. It then **packs the board**: `fillBlanks` places a `Blank` filler tile on every remaining off-corridor land cell, leaving `SLIDE.emptySlots` (~3) as passable holes. Finally it **scrambles by swapping** `SLIDE.scrambleSwapRatio` (~70%) of the movable rails into far-off blank tiles — teleporting rails into off-corridor slots so the track is visibly scattered (each rail goes to the farthest of a few sampled blanks); the `(type,orient)` multiset the corridor needs is preserved (no rotation in gameplay). **Solvability rests on Wilson's theorem (1974):** on a 2-connected, non-cycle graph with ≥2 holes or ≥2 identical tiles, every arrangement of the distinct tiles is reachable — and the board has ~55 identical blanks + 3 holes. A complete solver is infeasible at these sizes (it can never prove a board unsolvable), so the theorem preconditions — 2-connected `Empty` region, not a cycle, ≥2 blanks, ≥1 hole, multiset preserved — ARE the proof, asserted directly in the smoke test rather than via a runtime solver. `scramble(level, ratio)` is the swap-scramble. Grid size grows with level number. Tunables are in `src/constants.ts`: `GRID` (layout + obstacle density)/`SLIDE` (`scrambleSwapRatio` + `emptySlots`)/`COLORS` (palette), `TIMING` (train speed + hold durations), `VISUAL` for animation speeds.

### Input model (axis-locked multi-cell drag)

`src/input.ts` is drag-based:
- **Movable tile** (`Straight`/`Curve` rail, or `Blank` filler): press and **drag** — the drag locks to its **dominant axis** and the tile **slides straight as many cells as the drag distance allows**, stopping at the first barrier (terrain/other tiles/stations/border). A drag below `cell/4` is a tap (no move). On release, `desiredSteps` rounds the drag distance along the axis to whole cells, clamped to `slideDistance`, and `stepDrop(dir, steps)` commits an atomic `slideRail`. **No neighbor hints are drawn**: the renderer just draws the carried tile (base + rail, or base + blank marker) following the pointer, clamped to the cells it can actually slide through — so a blocked direction leaves the tile visibly stuck (the "area is taken, can't go there" feedback), and a drag with no net progress snaps back to the source on release.
- **Immovable cell** (`Terrain` / `Tunnel`): press does nothing. Terrain renders as a bolted plate and tunnels sit on it (both read as fixed anchors via the bolts).
- **SEND** button: manually launch the train to test the current track (incomplete → crash → back to Building). A complete path auto-launches regardless.

The input layer tracks a `slideDrag` flag, the live pointer, and the press start (`pressStart`); the renderer leaves the source cell empty and draws the carried rail nudged from its source cell while a slide drag is in progress.

### Rendering & layout

`src/layout.ts` `computeLayout()` maps the viewport to a centered map area + bottom panel, cell-sized to fit while respecting a touch-friendly floor (`GRID.minCellPx`). It is DPR-aware; `main.ts` applies `ctx.setTransform(dpr,0,0,dpr,0,0)` so all drawing uses CSS px. `src/render/shapes.ts` holds procedural drawing primitives; `renderer.ts` draws the field, `ui.ts` draws the panel/overlays. Layout is recomputed each frame (cheap) and on resize.

**Retro vector style:** the field is drawn directly at display/DPR resolution (no offscreen upscaling) using `src/render/shapes.ts` primitives — flat fills, black outlines, and blocky sprites. A global `time` (accumulated in `main.ts`) drives train smoke and crash animations; it is passed down through `renderer.draw` / `ui.draw` into the shape functions. The canvas is DPR-scaled in `main.ts` via `ctx.setTransform(dpr,0,0,dpr,0,0)`, so all drawing uses CSS px and stays crisp on dense screens. Empty movable tiles are drawn as framed squares with a **recessed well** (a darker inset fill, no frame) so empty holes read as obvious gaps next to occupied tiles. `Blank` filler tiles render as a tile base plus a small centered `drawBlankMark` square (no rail) — distinct from both holes and rail tiles. `drawRail` cannot render a Blank (it has no ports), so the renderer draws the marker instead and skips `drawRail` for Blanks. **Terrain is drawn as a bolted tile** (`drawTerrain`): a flat riveted plate (corner bolts + a center rivet) that reads as a fixed, immovable obstacle; Tunnels sit on terrain and draw only their track + dark portal mouths on top (the plate supplies the bolts).

### Audio

`src/audio.ts` `sfx` is a singleton synthesizing all sound effects via Web Audio (no asset files). The `AudioContext` is created lazily in `unlock()`, which **must** be called from a user gesture — `input.ts` calls it on the first `pointerdown`. Game-logic code never calls audio directly (keep logic pure); instead `input.ts` plays interaction sounds (move/send) and `main.ts` plays state-transition sounds (chug start/stop, crash, level-complete) by diffing `game.state` between frames. The mute toggle (`sfx.toggle()`) is wired to a panel button; `main.ts` passes `sfx.enabled` to the UI as the `muted` flag.

### Game loop

`src/loop.ts` is a `requestAnimationFrame` loop with clamped delta (max 0.1s to survive tab inactivity). It is variable-timestep, not fixed-step — gameplay code must be dt-based.

---
> Source: [alaypanov/mad-rails](https://github.com/alaypanov/mad-rails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
