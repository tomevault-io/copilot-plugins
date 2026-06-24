## photo-editor

> TypeScript · Vite · Zustand vanilla (state) · Canvas 2D (rendering, upgrading to WebGPU)

# Photo/Video Editor — Architecture Notes

## Stack
TypeScript · Vite · Zustand vanilla (state) · Canvas 2D (rendering, upgrading to WebGPU)
No framework. Final goal: C++ WASM core + WebGPU renderer.

> **Zustand note:** import from `zustand/vanilla` (not `zustand`). The default `zustand`
> export in v5 requires React. Use `createStore<T>()((set, get) => ...)` — note the curried call.

## Source layout

```
src/
├── engine/              Pure math — NO DOM, no store imports. Future WASM boundary.
│   └── viewport-math.ts  zoom/pan coordinate transforms
├── store/               Zustand slices. All mutable state lives here.
│   ├── viewport-store.ts  scale, offsetX/Y, canvasW/H + actions
│   └── playback-store.ts  video state + actions; owns <video> element as closure
├── renderer/            Drawing backend. Swap Canvas2D → WebGPU by changing one line.
│   ├── types.ts          Renderer interface, RenderSource, Transform
│   └── canvas2d-renderer.ts
├── input/               DOM events → store actions. No state, no DOM reads.
│   ├── pointer-input.ts  mouse/wheel → viewportStore
│   └── keyboard-input.ts keyboard shortcuts → store actions
├── ui/                  Dumb components: subscribe to stores, fire actions, no logic.
│   ├── toolbar.ts
│   ├── playback-bar.ts
│   └── status-bar.ts
├── app.ts               Thin root. Owns canvas element, DPR, render loop. Wires modules.
└── main.ts              Entry: grabs DOM refs → App constructor. ~20 lines.
```

## Key invariants

**1. Engine functions are pure.**
`src/engine/` has zero imports from stores, services, or DOM APIs. Functions take plain
data in, return plain data out. This is the WASM boundary — these will be replaced by
C++ equivalents compiled to WASM without touching any other layer.

**2. Stores own all state.**
Components never hold mutable state. They subscribe to stores for reads and call store
actions for writes. No passing callbacks down through constructor chains.

**3. UI components are dumb.**
`src/ui/` files contain no business logic. They bind DOM events to store actions and
subscribe to stores to update the DOM. They can import stores directly.

**4. App.ts owns the render loop exclusively.**
- While playing: continuous `requestAnimationFrame` loop (`startLoop` / `stopLoop`).
- While paused: demand-driven single-frame (`scheduleRender`), triggered by store subscriptions.
- `scheduleRender` is a no-op while the loop is running — no double renders.

**5. DPR scaling lives only in `App.drawFrame`.**
The renderer always works in logical (CSS pixel) coordinates. `ctx.setTransform(dpr, ...)` is
applied once per frame in App before calling `renderer.render()`. The renderer knows nothing
about device pixel ratios.

**6. The `<video>` element is a private implementation detail.**
It lives as a closure variable inside `playback-store.ts`. It is never part of the public
store state. The only place it is read externally is `App.drawFrame()` via `getElement()`.

## Data flow

```
User gesture / keyboard
  → input handler
  → store.action()
  → store notifies subscribers
  → App.scheduleRender() or UI component updates DOM

Video playing
  → continuous rAF loop in App
  → App.drawFrame() reads store state
  → renderer.render(source, transform, canvasW, canvasH)
```

## Render pipeline

```
App.drawFrame()
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)   ← DPR applied here, once
  Canvas2DRenderer.render(source, transform, logicalW, logicalH)
    fillRect background
    fillRect checker pattern
    fillRect shadow
    drawImage source
    strokeRect frame border
```

## State stores

### viewportStore
| Field    | Type   | Meaning                                          |
|----------|--------|--------------------------------------------------|
| scale    | number | Current zoom (1.0 = 100%)                        |
| offsetX  | number | Image top-left X in logical canvas coords        |
| offsetY  | number | Image top-left Y in logical canvas coords        |
| canvasW  | number | Canvas logical width (updated on resize)         |
| canvasH  | number | Canvas logical height (updated on resize)        |

### playbackStore
| Field       | Type   | Meaning                              |
|-------------|--------|--------------------------------------|
| isPlaying   | bool   | Video is actively playing            |
| currentTime | number | Current position (seconds)           |
| duration    | number | Total length (seconds)               |
| srcWidth    | number | Video frame width in pixels          |
| srcHeight   | number | Video frame height in pixels         |

## Adding features

### New keyboard shortcut
Add a key → action entry in `src/input/keyboard-input.ts`.

### New toolbar button
Add the HTML element, then wire it in `src/ui/toolbar.ts` with a store action call.

### New image filter
1. Add filter state to a store (or new `src/store/filter-store.ts`)
2. Write the math as a pure function in `src/engine/`
3. Apply in `Canvas2DRenderer.render()` — or as a WebGPU compute shader when that lands

### Swap to WebGPU renderer
1. Implement `src/renderer/webgpu-renderer.ts` satisfying the `Renderer` interface
2. Change one line in `app.ts`: `new Canvas2DRenderer(ctx)` → `new WebGPURenderer(device)`

### Add image support alongside video
`playbackStore` currently owns a `<video>` element. To support images too:
- Create `src/store/source-store.ts` with a union type `VideoSource | ImageSource`
- Both expose `element: CanvasImageSource`, `srcWidth`, `srcHeight`
- `App.drawFrame` reads from `sourceStore` instead of `playbackStore` for dimensions/element
- `playbackStore` becomes video-specific metadata (time, duration, play state)

---
> Source: [dharness/photo-editor](https://github.com/dharness/photo-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
