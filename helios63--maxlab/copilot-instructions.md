## maxlab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (http://localhost:3000)
npm run build      # Production build
npm run generate   # Static site generation
npm run preview    # Preview production build
```

No test suite or linter is configured.

## Architecture

This is a **Nuxt 4 + p5.js creative coding playground** — a single-page app that renders generative art sketches full-screen.

### Sketch system

Each sketch lives in `app/sketches/*.ts` and exports a **factory function** with this signature:

```ts
export default function mySketch(container: HTMLElement) {
  return (p: any) => {
    p.setup = () => { ... }
    p.draw  = () => { ... }
    // Expose optional imperative hooks:
    p.resetDrawings = () => { ... }
    p.setHue = (hue: number) => { ... }
  }
}
```

`P5Sketch.vue` (`app/components/`) dynamically imports p5 and calls `new P5(factory(container), container)`. It exposes `reset()` and `setHue()` to the parent via `defineExpose`.

`app/pages/index.vue` owns the sketch registry (an array of `{ name, factory }` objects) and the nav menu for switching between them. Only the active sketch's `reset` button and color picker are shown (currently wired only for sketch index 0).

### Sketches

| File | Description |
|---|---|
| `timeFlowers.ts` | Clock rendered as p5 flowers arranged in a pixel-font grid; flowers spiral in/out on digit change, sky color tracks time of day |
| `timeGrid.ts` | Three transparent additive-blended grids (H/M/S) fill cells to represent current time |
| `timePhysics.ts` | Physics clock using Matter.js — shapes fall and stack; count of large/medium/small shapes = hours/minutes/seconds |
| `bouncingSquare.ts` | Bouncing squares with AABB collision; user can draw pixels with the mouse |

### Adding a sketch

1. Create `app/sketches/mySketch.ts` following the factory pattern above.
2. Import it in `app/pages/index.vue` and add `{ name: '...', factory: mySketch }` to the `sketches` array.

### Key dependencies

- **p5** (`^2.2.3`) — rendering; loaded lazily inside `P5Sketch.vue`
- **matter-js** (`^0.20.0`) — physics engine used only by `timePhysics.ts`
- **@nuxtjs/tailwindcss** — utility CSS; global styles in `app/assets/css/main.css`

---
> Source: [helios63/MaxLab](https://github.com/helios63/MaxLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
