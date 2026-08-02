## system-design-simulator

> Guidance for AI agents (and humans) working in this repo.

# CLAUDE.md

Guidance for AI agents (and humans) working in this repo.

> **Read `AGENTS.md` first.** This is Next.js 16 — APIs and conventions differ from older versions. Check `node_modules/next/dist/docs/` before writing framework code.

## What this is

SystemForge — an open-source system-design interview simulator. Drag infrastructure components onto a React Flow canvas, wire them, simulate production-scale traffic (Kahn's topological sort), and get scored across 5 interview dimensions. 100% client-side; state persists to `localStorage`. No backend.

## Commands

```bash
npm run dev      # dev server (http://localhost:3000)
npm run build    # production build — also runs tsc; must pass before pushing
npm run lint     # eslint
npx tsc --noEmit # type-check only
```

There are no unit tests; verify changes by building and exercising flows in the browser.

## Tech stack

Next.js 16 (App Router, single static `/` route) · React 19 · TypeScript · @xyflow/react v12 (ReactFlow) · Zustand v5 (persisted) · Tailwind v4 · base-ui dialogs/primitives · framer-motion · perfect-freehand (pen) · html-to-image (export). No new runtime deps without good reason.

## Architecture map

```
src/
  app/            App Router entry, layout, globals.css (dark-only theme)
  components/
    canvas/       DesignCanvas (ReactFlow host), nodes/ (Component, Text), edges/, PenOverlay/PenToolbar, CanvasTabBar
    panel/        RightPanel + Props/Sim/Score/Capacity/Tradeoffs tabs
    sidebar/      Sidebar: ComponentPalette, ProblemSelector, LearningPath
    layout/       AppShell (orchestrator + keyboard shortcuts), TopBar, SupportFAB
    interview/    InterviewBar, phase panel, start dialog
    dialogs/      ModalShell (shared modal: focus trap/Escape/scroll) + Save/Load/Confirm/Support/Create*
    ui/           shadcn-style primitives, Toast
  data/           components.ts (30 specs), problems.ts (35), conceptLibrary.ts,
                  interviewData.ts, tradeoffCards.ts (21), learningPath.ts
  engine/         simulator.ts (traffic sim), constants.ts
  scoring/        scorer.ts + rules/ (scalability, availability, latency, cost, tradeoffs — 20 pts each)
  store/          zustand stores (see below)
  lib/            exportCanvas, loadReference, icons, utils
  types/          shared interfaces
```

## Key invariants — don't break these

**Simulation engine (`engine/simulator.ts`).** Called as `runSimulation(componentNodes, allEdges, requestsPerSec)` — text nodes are filtered out but ALL edges are passed, so edges may reference non-component nodes; the engine must skip edges whose source/target isn't a known component node. Entry nodes = in-degree 0 **with** outgoing edges (a fully disconnected node is NOT an entry and must not receive traffic). Sanitize `maxQPS`/`replicas` (finite, positive) before use. Reported throughput never exceeds offered load. Async edges (`edge.data.async`) are excluded from user-facing latency. LBs split traffic evenly; other nodes fan out 100% to each child (intentional).

**Scoring (`scoring/`).** `scorer.ts` builds a shared `ScoringGraph` (cleaned adjacency + reachable-from-entry set) once and passes it to every rule. Presence checks must require reachability — placing a component without wiring it earns no points (with feedback saying so). Each category rule must total **exactly 20** max and never go negative; verify the arithmetic if you touch a rule.

**Stores (`store/`).** Every persisted store uses `version: 1`, `skipHydration: true`, a no-op `migrate`, and `safeLocalStorage` (from `safeStorage.ts`, swallows QuotaExceeded + toasts). Hydration is deferred: `hydration.ts` exports `rehydrateAllStores()` and `useHasHydrated()` — call after mount to avoid SSR mismatch. `canvasStore` persists the active tab with empty nodes/edges (live copies live at the top level; reconstructed on rehydrate) and strips runtime fields (`utilization`/`status`/`isBottleneck`). It also has unpersisted undo/redo history (`undo`/`redo`/`canUndo`/`canRedo`, 50 entries, pushed before mutation) and `deleteEdge(id)`. `interviewStore` timer is timestamp-based (`startedAt`/`accumulatedMs`) so it survives background-tab throttling and refresh — never reintroduce tick-counting.

**Persistence schema.** `SerializedEdge` must carry `data` (label/protocol/async) or edge metadata is lost on save/load. Export/import use a unified envelope `{ schemaVersion, name, problemId, nodes, edges, strokes }`; `importDesign` validates structurally and returns `{ ok, error? }`.

**Canvas/UI.** `nodeTypes`/`edgeTypes` are module-level (never inline — causes remounts). Reference tabs (`tab.readOnly`) must gate dragging/connecting/dropping/delete. Keyboard shortcuts must no-op while typing in inputs; there is one delete path (`deleteKeyCode={null}` on ReactFlow + the AppShell handler covering node AND edge selection). Dialogs go through `ModalShell`. Touch: hover-only affordances are invisible on coarse pointers (Tailwind v4 gates `hover:` behind `@media(hover:hover)`) — gate visibility on `useIsCoarsePointer()` instead.

## Data conventions

- Component `id`s referenced in `problems.ts` reference solutions, `conceptLibrary.ts`, and `learningPath.ts` must exist in `components.ts`. A single reference solution must NOT reuse the same `componentId` twice (the loader wires edges by componentId).
- `learningPath.ts` prerequisites must be concepts taught by a strictly **earlier** problem in path order.
- All 35 problems must have entries in `interviewData.ts` and a learning-path tier.
- Content teaches interview candidates — every formula, figure, API shape, and real-world attribution must be correct.

## Conventions

- Dark theme only (`<html class="dark">`); there is no theme toggle. Use zinc-* palette; sub-11px labels use `text-zinc-400`+ for contrast.
- Temp/scratch files: keep them out of the repo.
- Commit messages: do NOT add Claude/AI attribution or co-author trailers.

---
> Source: [vijaygupta18/system-design-simulator](https://github.com/vijaygupta18/system-design-simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
