## db-flow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands run from the `db-flow/` directory:

```bash
npm run dev       # Start dev server (Vite)
npm run build     # Type-check + production build
npm run lint      # ESLint
npm run preview   # Preview production build locally
```

There are no tests configured yet.

## Workflow

**Na elke afgeronde taak moet er een git commit worden gemaakt.** Dit is verplicht en geldt voor elke wijziging, hoe klein ook.

Een `prepare-commit-msg` hook is aanwezig in `.git/hooks/` en vult automatisch een basis commit message in als er geen wordt meegegeven.

Gebruik altijd een beschrijvende commit message die uitlegt *wat* er is veranderd en *waarom*. Voeg altijd de volgende co-author toe:

```
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

## Architecture

This is a React + ReactFlow application for visualizing database schemas as Entity-Relationship Diagrams. The app supports two graph modes (`"erd"` and `"pipeline"`), though only ERD is actively built out.

### State

All graph state lives in a single Zustand store at [db-flow/src/store/graphStore.ts](db-flow/src/store/graphStore.ts). It holds `nodes`, `edges`, `mode`, and `selectedNodeId`, and exposes mutation helpers used throughout the app. ReactFlow change callbacks (`onNodesChange`, `onEdgesChange`) are wired directly through this store.

### Type system

[db-flow/src/graph/types.ts](db-flow/src/graph/types.ts) defines the core domain types:
- `AppNode` / `AppEdge` — typed wrappers around ReactFlow's `Node`/`Edge`
- `ERDNodeData` — table name + `Column[]` (with `isPK`, `isFK`, `isUnique`, `isNullable` flags)
- `PipelineNodeData` — stub for the pipeline mode
- `RelationEdgeData` — `relationType` (`"one-to-one"` | `"one-to-many"` | `"many-to-many"`) + optional `fromColumn`/`toColumn`
- `GraphMode` — `"erd"` | `"pipeline"`

### ERD rendering

- **TableNode** ([db-flow/src/erd/nodes/TableNode.tsx](db-flow/src/erd/nodes/TableNode.tsx)) — renders a card with a dark header (table name) and a row per column. PK rows are amber-tinted, FK rows are indigo-tinted. Single source handle (right) and target handle (left).
- **RelationEdge** ([db-flow/src/erd/edges/RelationEdge.tsx](db-flow/src/erd/edges/RelationEdge.tsx)) — smooth-step path with a centered label badge showing the relation type (1:1, 1:N, N:M) and optional column names. Color-coded by relation type.

Do not put styling direct in the components.

### Layout

[db-flow/src/graph/layout.ts](db-flow/src/graph/layout.ts) wraps dagre for auto-layout. ERD uses top-to-bottom (`rankdir: "TB"`), pipeline uses left-to-right (`"LR"`). Called from the Toolbar's "Auto-layout" button via `layoutGraph(nodes, edges, mode)`.

### App shell

[db-flow/src/app/App.tsx](db-flow/src/app/App.tsx) composes the full layout: a fixed top bar + a `ReactFlowProvider` wrapping the canvas. `ReactFlowProvider` is required so that child components (like `Toolbar`) can call `useReactFlow()`.

Node and edge type registries (`nodeTypes`, `edgeTypes`) are defined at module level in App.tsx — keep them stable references (not inline objects) to avoid ReactFlow re-mounting nodes on every render.

### Pipeline mode

`PipelineNode` and `pipeline/rules.ts` are stubs — not yet integrated into the main graph or store.

---
> Source: [LuuckyG/DB-Flow](https://github.com/LuuckyG/DB-Flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
