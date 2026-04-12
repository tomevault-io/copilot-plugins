## lego-langgraph

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

```bash
# Development
npm run dev              # Start dev server on port 5173
npm run build            # TypeScript check + Vite build
npm run types            # TypeScript type check only
npm run lint             # ESLint
npm run preview          # Preview production build

# Testing
npm run test             # Vitest watch mode
npm run test:ui          # Vitest UI mode
npm run test:run         # Run tests once (watch=false)
npm run test:e2e:install # Install Playwright browsers
npm run test:e2e         # Run E2E tests
npm run test:e2e:ui      # Playwright UI mode
```

## Code Style

- **File naming**: `PascalCase.tsx` for components, `camelCase.ts` for utilities
- **Exports**: Prefer named exports; default only for React components
- **Conventional Commits**: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`
- **TypeScript**: Strict mode enabled (`noUncheckedIndexedAccess`, `noUnusedLocals`, `noUnusedParameters`)

## Test Conventions

- **Arrange-Act-Assert** structure
- **Reset store state** in `beforeEach` each test
- Store tests: `useGraphStore.getState().clearGraph()` before each test
- Mock external deps (localStorage, fetch)

## Architecture

**LangGraph Visual Modeler** - A deterministic, explainable visual modeling environment for LangGraph applications. No backend execution required.

### Tech Stack
- **Frontend**: React 18 + TypeScript + Vite
- **Canvas**: React Flow (node-edge graph modeling)
- **State**: Zustand (graph, simulation, UI stores)
- **Storage**: localStorage with auto-save

### Core Concepts

**7 Node Types**: `Start` | `LLM` | `Tool` | `Router` | `Reducer` | `LoopGuard` | `End`

**Simulation Engine**: Deterministic step-by-step execution with cycle detection, edge firing explanations, and immutable state snapshots. Mock execution for MVP (no real LLM calls).

**Data Models** ([`src/models/`](src/models/)):
- [`graph.ts`](src/models/graph.ts) - `GraphModel`, `GraphNode`, `GraphEdge`, `validateGraph()`
- [`state.ts`](src/models/state.ts) - `StateSchema` (field definitions)
- [`simulation.ts`](src/models/simulation.ts) - `ExecutionTrace`, `SimulationStep`, `GraphState`

**Store Architecture** ([`src/store/`](src/store/)):
- [`graphStore.ts`](src/store/graphStore.ts) - Nodes, edges, CRUD, selection, localStorage persistence
- [`stateStore.ts`](src/store/stateStore.ts) - State schema fields
- [`simulationStore.ts`](src/store/simulationStore.ts) - Trace, playback state, current step

**Feature Structure**:
- [`features/canvas/`](src/features/canvas/) - React Flow canvas, node renderers, inspector panels
- [`features/state/`](src/features/state/) - State schema editor, field CRUD
- [`features/sim/`](src/features/sim/) - Step controls, trace list, state diff viewer
- [`features/io/`](src/features/io/) - JSON export/import, toolbar

**Services**:
- [`SimulationEngine`](src/services/SimulationEngine.ts) - Core simulation logic with deterministic edge ordering
- [`conditionEvaluator`](src/services/conditionEvaluator.ts) - Edge condition parsing and explanation generation

### App Layout

3-column grid ([`App.tsx`](src/app/App.tsx)):
- **Left**: State Schema Panel
- **Center**: Canvas + Toolbar + Simulation Controls
- **Right**: Node/Edge Inspector (when selected)

### Path Aliases

`@/*` → `./src/*` (configured in [`vite.config.ts`](vite.config.ts) and [`tsconfig.json`](tsconfig.json))

### Determinism Rules

1. Edge execution order: sorted by edge ID (ascending)
2. Router: fires **first** matching edge only
3. LoopGuard: blocks all if no conditions match
4. Cycle detection: tracks visited nodes in current path

### Validation

Graph validation ([`graph.ts:85`](src/models/graph.ts#L85)):
- Requires at least one `Start` node
- Requires at least one `End` node
- Checks for orphaned edges
- Detects duplicate node IDs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/htooayelwinict)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/htooayelwinict)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
