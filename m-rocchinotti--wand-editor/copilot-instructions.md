## wand-editor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git & GitHub Workflow

Always commit changes locally and push to GitHub after each session or meaningful change.

- Remote: `https://github.com/M-Rocchinotti/wand-editor.git`
- Branch: `master`
- Write clean, descriptive commit messages
- Push after every session so there's always a saved version to revert to

## Running the app

No build step. Open `wall-designer-pro.html` directly in any browser. Two CDN dependencies load at runtime:
- `jszip 3.10.1` — save/load ZIP files
- `jspdf 2.5.1` — PDF export

## Architecture

Single self-contained HTML file: `<style>` blocks → `<body>` markup → one large `<script>` block (~2200 lines).

### Global state

| Variable | Type | Purpose |
|---|---|---|
| `W` | object | Wall config: `{len, h, d, col: {wall, side, floor}}` (mm) |
| `LT` | object | Light source: `{x, y, on, i}` |
| `pages` | array | Array of `{name, objs[]}` — multi-page |
| `curP` | number | Active page index |
| `CAM` | object | Camera: `{az, el, zoom, panX, panY}` |
| `sel` | number\|null | Selected object ID |
| `multiSel` | Set | Multi-selected object IDs |
| `TC` | object | Texture cache: `id → Image` |
| `MESHES` | object | Imported 3D mesh data keyed by meshKey |

### Object schema

Every placed element: `{id, t, x, y, w, h, d, color, label, _p, texUrl, rd, wz, meshKey}`

- **Wall objects**: `x` = left edge on wall, `y` = bottom edge (from floor), `wz` = offset from wall face
- **Floor objects**: `x` = left edge, `rd` = distance from wall into room, `y` is always 0

`isFloor(t)` determines which type a given element `t` belongs to.

### Coordinate system & projection

World space is mm. `iso(wx, wy, wz)` projects world coords to screen `[px, py]` using `CAM.az`/`CAM.el` spherical rotation. Inverse: `screenToWall(px, py, targetWZ)` and `screenToFloor(px, py)`.

`layout()` recomputes `SC` (world→screen scale), `OX`/`OY` (canvas origin) on every draw to fit the current wall into the viewport.

### Render pipeline

`draw()` calls in order:
1. `layout()` — resize canvas, recompute SC/OX/OY
2. `drawFloor()` — floor quad with gradient
3. `drawWall()` — wall box (6 faces, visibility culled)
4. Sort all objects by depth (`rd+d` or `wz+d`) then paint
5. Per object: `drawWallObj`, `drawFloorObj`, or `drawMeshObj`
6. Multi-select outlines, lock icons
7. `drawGhost()` — placement preview
8. `drawRulers()` — horizontal/vertical gap rulers
9. `drawSelDims()` — interactive dimension overlays for selected object
10. `flushDims()` — inject editable `<input>` widgets over dim lines

`wallBox(x,y,w,h,d,fc,tc,sc,p,ghost,img,wallZ)` and `floorBox(...)` are the core primitives — they draw a 3D box with face-visibility culling and optional texture mapping.

### Element type system

`ETYPES` array: each entry `{t, label, w, h, d, color, cat, [meshKey], [imported]}`.

Categories: `'Wall furniture'`, `'Wall structural'`, `'Floor furniture'`, `'Floor structural'`, `'Lighting'`.

`FLOOR_CATS` determines which categories are floor objects. Imported 3D models are appended to `ETYPES` and `IMPORTED` at runtime.

### 3D mesh import

Parsers for OBJ, STL, GLB, PLY → `rawTris[]` → `processMesh()` normalises to unit cube, stores in `MESHES[key]`. `buildMeshCanvas(o)` renders to an offscreen `<canvas>` and caches it in `MESH_RENDER_CACHE[o.id]` keyed by a string encoding all relevant state.

### Save / load

`.wallstudio` = JSON (schema version `v:6`). If meshes present, wraps in ZIP with `.wallstudio.zip` extension. Shareable URL uses `v:7` base64-encoded JSON fragment.

`applyScene(txt)` is the single entry point for loading any saved state.

### Undo / redo

`snapshot()` serialises full state to JSON and pushes to `undoStack` (capped at 60). `undo()`/`redo()` call `applySnapshot()` which does a full state restore including rebuilding tabs, dots, and redrawing.

### i18n

`LANGS` object holds 7 languages (en, de, es, it, ru, zh, fr). `t(key, ...args)` looks up the active `curLang`. String keys are used throughout; `data-i18n` attributes on static HTML are updated by `applyLang()`. When adding new UI text, add a key to every language block in `LANGS`.

### 2D floor plan / wall elevation

Toggled via `toggleFloorPlan()`. Draws onto `#fp-cv` (separate canvas layered over the 3D view) using `drawFloorPlanView()` or `drawWallElevationView()`. `fpDimLine()` draws annotated dimension lines. PDF export renders these views to an offscreen canvas via `exportPDF()`.

---

## Behavioral Guidelines

Derived from Andrej Karpathy's observations on LLM coding pitfalls. Bias toward caution over speed — use judgment on trivial tasks.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

---

**Guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, clarifying questions come before implementation rather than after mistakes.

---
> Source: [M-Rocchinotti/wand-editor](https://github.com/M-Rocchinotti/wand-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
