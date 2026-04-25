## plato

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# From the monorepo root (../):
npm run build   # TypeScript type-checking (tsc -b)
npm run bundle  # Webpack bundle — required for the app to pick up changes
```

The HTML loads `../dist/plato/bundle.js` (webpack output). Running only `tsc -b` does **not** update the bundle.

## Architecture

**Plato** is an interactive 3D polyhedra viewer — a static web app where users can view, rotate, and draw on Platonic solids and related shapes.

### Entry Points
- `index.html` — Two stacked canvases: `#mainCanvas` (3D) and `#textureCanvas` (drawing overlay)
- `src/ts/index.ts` — Dynamically imports `app.ts`, initializes with `stellated_dodecahedron`

### Core Modules

**`src/ts/app.ts`** — Main application controller. Handles model loading, UI events, drawing with mirror symmetry, color/pen selection, and canvas lifecycle. Uses Vue 3 (minimal — just the model selector dropdown) and localStorage for caching loaded models.

**`src/ts/webgl_renderer.ts`** — Wraps Three.js. Three rendering passes:
1. `main_material` — 3D model with texture mapping
2. `mirror_material` — Reflection plane geometry (cyan overlay)
3. `overlay_material` — Drawing canvas as a fullscreen quad

**`src/ts/glsl.ts`** — Raw GLSL shader strings. Directional lighting via dot-product normal shading; texture sampling with alpha blending.

**`src/ts/webgl_utils.ts`** — Matrix math helpers (`gl-matrix`) and geometry conversion: `model_to_geometry()` converts model JSON → Three.js `BufferGeometry`, `matrix_uniforms()` computes projection/view/rotation matrices.

### Model Format

3D models live in `static/models/*.json`:
```typescript
{
  vertices: number[],       // XYZ triplets
  normals?: number[],       // surface normals
  texture?: number[],       // UV coordinates
  colors?: number[],        // RGBA vertex colors
  mirrors?: number[],       // mirror plane geometry
  mirror_normals?: number[] // mirror plane normals
}
```

### Dependencies
- `three` — WebGL rendering abstraction
- `gl-matrix` — 3D matrix/vector math
- `@oded/tsutils` — local monorepo package (color parsing, floodfill)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/odedbadt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
