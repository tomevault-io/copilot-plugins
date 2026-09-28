## codereel

> Turns code into an animated video. React + TypeScript + Vite + Tailwind v4.

# CodeReel

Turns code into an animated video. React + TypeScript + Vite + Tailwind v4.
**GIF export is real** — `ExportModal.tsx` drives `lib/export/gif.ts`, which steps
`progress` 0→1, draws each WebGL frame on demand, and encodes with `gifenc`. The
modal has a config step: the user picks **resolution** (longest edge, up to the
canvas's native size — never upscaled) and **frame rate**, with a live size
estimate.
(MP4/WebM are not wired up yet; Remotion/headless Chromium stays a future option
for those.)

The **WebGL renderer (`WebGLScene.tsx`, React Three Fiber) is the sole renderer.**
The old DOM/CSS renderer (`PreviewCanvas.tsx`) has been retired — CSS 3D can't be
rasterized in-browser, so it couldn't be exported.

## Core architectural rule (important)

The preview renders as a **pure function of `progress` (0→1)**. `WebGLScene`
takes `progress` as a prop; `usePlayback` owns the wall-clock and feeds it in.
Export swaps that source: it feeds discrete `progress` values instead.

**Any new visual effect must be a pure function of `progress` — no wall-clock in
anything drawn into a frame** (no CSS `@keyframes`/`animation`/`transition`, no
`requestAnimationFrame`/`setInterval`/`Date.now()`/`performance.now()`, no
unseeded `Math.random()`). Derive motion from `progress`, e.g. `x = progress * k`.

This is what lets export reuse the exact same rendering code with **no rework**.

## Diff view (steps mode)

`Settings.diffMode` (top-level, default `false`; toggle in `CodePanel`) turns each
step into a **git-style merged diff against the previous step**: removed lines are
tinted red with a `−` gutter mark + strikethrough, added lines green with `+`,
unchanged lines plain, and line numbers follow the new-file numbering (skipping
removed rows). Step 0 has no predecessor, so it renders plain.

- **`diffLines(prev, next)`** (`src/lib/diff.ts`) produces the merged
  `same`/`added`/`removed` sequence. (`diffLineStatus` — added-only — still powers
  the hero-line spotlight.)
- The merged view is rasterized by `drawCard`/`paintCard` via an optional per-line
  `diff` status array; the **transition** into a diffed step animates the markers
  in (removed struck first, added fade+slide in) through the `animT` param on a
  dynamic texture (same pattern as the console typewriter) — still a pure function
  of `progress`, so export is unaffected.
- Diff is steps-only (needs a before/after). In sequence mode the toggle is a no-op.

## Key files

- `src/components/WebGLScene.tsx` — the renderer (all reveals/transitions/3D, R3F).
  Snapshots share one max-sized texture box so shader transitions line up; the
  extruded `slab` is scaled per-frame to each snapshot's actual card size
  (`cardSizes`) so it doesn't overhang a shorter snapshot as a phantom card.
- `src/lib/codeTexture.ts` — rasterizes a snapshot to a canvas→texture (`drawCard`);
  also paints the diff view + its per-line reveal animation
- `src/lib/diff.ts` — LCS line diff: `diffLineStatus` (added-only) + `diffLines` (merged)
- `src/lib/export/gif.ts` — GIF exporter (frame-steps `progress`, composites
  background + canvas + brand overlay, encodes via `gifenc`)
- `src/lib/timeline.ts` — stepped-mode timeline (`buildTimeline`, `locate`), pure
- `src/lib/usePlayback.ts` — the rAF clock (live preview); export bypasses it and
  drives frames via `App`'s `renderAt` → R3F `advance()` (no rAF, so a backgrounded
  tab doesn't stall the export)
- `src/lib/types.ts` — `Settings` + option lists

## 3D effects roadmap

Tier 1/2/3 plan, current status, and how Remotion fits (it can come later with no
extra work if the rule above is kept): see **[`docs/3d-roadmap.md`](docs/3d-roadmap.md)**.

**Tiers 1 and 2 are complete.** Tier 1: hero-card perspective tilt (8-direction
pad), 3D flip-up reveal, `flip3d` step transition, depth-of-field (`dof`),
parallax backdrop (`parallax`). Tier 2: floor reflection (`reflection`), accent
bloom (`bloom`), per-token cascade ("Tokens" motion) + light-sweep (`sweep`).
Next up: Tier 3 (React Three Fiber — separate WebGL renderer, bigger lift).

## Commands

- `npm run dev` — Vite dev server
- `npm run build` — `tsc -b && vite build`
- `npm run lint` — oxlint

---
> Source: [eddiejaoude/CodeReel](https://github.com/eddiejaoude/CodeReel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
