## frameworkx

> Split-screen developer tool: Monaco editor + instrumentation iframe preview (left 30%) + live framework internals visualizer (right 70%).

# FrameworkX — Claude Code Context

## What This Is
Split-screen developer tool: Monaco editor + instrumentation iframe preview (left 30%) + live framework internals visualizer (right 70%).
Instrumentation runs inside a sandboxed iframe, sends postMessage events to the host, normalized into
Zustand stores that drive React Flow visualizations.

## Commands
```bash
npm run dev       # Vite dev server
npm run build     # tsc -b && vite build
npm run lint      # ESLint 9.x flat config
npm run preview   # Cloudflare Workers local preview via wrangler
npm run deploy    # Deploy to Cloudflare Workers via wrangler
npx tsc --noEmit  # Type-check only — run before every commit
```

## Stack (locked)
| Layer | Choice |
|---|---|
| App shell | Vite 7 + React 19.1 + TypeScript 5.7 |
| Editor | `@monaco-editor/react` 4.7 — read-only Monaco, preset examples only |
| Compilation | `@swc/wasm-web` (React/TSX → ESM), `@vue/compiler-sfc` (Vue SFC → ESM) |
| Visualization | `@xyflow/react` 12.x + Framer Motion 11.x |
| State | Zustand 5.x |
| Styling | Tailwind CSS 4.x via `@tailwindcss/vite` — CSS-first, no `tailwind.config.js` |
| Linting | ESLint 9.x flat config (`eslint.config.js`) + Prettier 3.x |

## Architecture (locked)
- One instrumentation iframe (visible, bottom 40% of left panel) — `sandbox="allow-scripts"` only, no `allow-same-origin`
- Iframe shell HTML built once per framework switch via `srcdoc`; code updates via postMessage `{type:'fx:update'}`
- Probe injection order is non-negotiable: probe `<script>` runs BEFORE importmap BEFORE framework CDN module
- postMessage flow: `iframe → host addEventListener → adapter → bridgeStore → visualizer`
- SWC WASM initializes eagerly on app load, not on first keystroke
- No React Router — single screen app

## Key Files
```
src/types/events.ts              # Canonical postMessage contracts — touch this first for new event shapes
src/types/index.ts               # Barrel export
src/store/editorStore.ts         # framework, code per framework, activeExample
src/store/bridgeStore.ts         # fiberTree, vueTree, depGraph, raw events[]
src/store/visualizerStore.ts     # activeTab, selectedNodeId
src/probes/react-probe.ts        # Runs INSIDE instrumentation iframe (not host bundle)
src/probes/vue-probe.ts          # Runs INSIDE instrumentation iframe
src/compiler/swc.ts              # compileReact() — SWC WASM, TSX → ESM
src/compiler/vue.ts              # compileVueSFC() — parse → compileScript → compileTemplate → attach render fn
src/adapters/react/              # Raw fiber → normalized FiberNode tree
src/adapters/vue/                # Raw Vue events → VueComponentNode + DepGraph
src/components/Shell/            # Layout, toolbar, framework toggle
src/components/Editor/           # EditorPlayground (Monaco + InstrumentationIframe)
src/components/Visualizer/       # React Flow panels (TreeVisualizer, ReactivityGraph)
```

## Type Conventions
- `FRAMEWORKS` const object is the single source of truth for `FrameworkType` — add frameworks there only
- Discriminate postMessage events on `source`: `'react-probe'` | `'vue-probe'`
- `FiberNode.didRerender` — per-commit boolean, reset each cycle, drives flash animation
- `FiberNode.renderCount` — cumulative, drives heatmap color

## Vue Compiler Constraints
- **`inlineTemplate: false` is mandatory** in `compileScript`. With `inlineTemplate: true`, setup() returns the render fn → Vue runtime sets `instance.render = fn` and never sets `instance.devtoolsRawSetupState` → dep→name matching in the probe fails.
- **`isProd: false` is mandatory** so Vue activates `__DEV__` code paths (devtools hooks, `devtoolsRawSetupState`).
- Template must be compiled separately via `compileTemplate` with `bindingMetadata: scriptResult.bindings` passed through, so the template compiler uses `$setup.CounterSection` (direct access) instead of `resolveComponent(...)`.

## Vue Probe Notes
- `app:init` fires AFTER all `component:added` events — use it as the signal to emit the initial tree.
- `_cycleUids` pattern: `component:updated` events in the same tick are batched in a Set, flushed in `setTimeout` callback.
- `instance.devtoolsRawSetupState` contains raw (unproxied) setup return: `{count: RefImpl, ...}`. Use this for dep→name mapping. Do NOT use `instance.setupState` (wrapped in proxyRefs — unwraps refs on GET, losing the Dep objects).
- Vue CDN must use `?dev` suffix (e.g. `esm.sh/vue@3.5.x?dev`) to activate devtools hooks. Without it, `component:added` and `component:updated` never fire.
- Vue starter code uses `h()` render functions (no runtime template compiler in the CDN build).

## Design Tokens (defined as CSS custom properties in `src/index.css`)
- `--color-bg-base: #F1F5F9` / `--color-bg-surface: #FFFFFF` / `--bg-surface-border: rgba(0,0,0,0.08)`
- `--color-accent: #00D4FF` / `--color-text-primary: #0F172A` / `--color-text-secondary: #64748B`
- Rerender scale: neutral `#CBD5E1` → yellow `#EAB308` → orange `#F97316` → red `#EF4444`
- Vue colors: ref `#3B82F6` / computed `#8B5CF6` / component `#10B981`
- Fonts: `Geist Sans` (UI), `JetBrains Mono` (code)

## Session Conventions
- Conventional commits (`feat:` `fix:` `chore:` `refactor:`) — stop at logical points for user to commit
- Give commands to run, don't execute them directly (user runs npm/npx)
- Run `npx tsc --noEmit` before declaring any block done
- Light theme — FrameworkX UI is light
- No over-engineering — hit done condition, move on

## Block Progress
- [x] Block 1 — Scaffold + Tailwind + types + stores + shell
- [x] Block 2 — EditorPlayground (Monaco read-only editor + InstrumentationIframe, 30/70 layout)
- [x] Block 3 — Instrumentation iframe + probe injection
- [x] Block 4 — postMessage bridge + adapters
- [x] Block 5 — React component tree + re-render highlighting
- [x] Block 6 — Vue reactivity graph
- [ ] Block 7 — Event log + preset examples
- [ ] Block 8 — Polish + smoke test

## Out of Scope for v0
Source maps, Vue component tree, lifecycle timeline, vDOM diff, resizable panes, playback,
shareable URLs, PNG/SVG export, comparison mode, collaboration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sharmaadityaHQ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
