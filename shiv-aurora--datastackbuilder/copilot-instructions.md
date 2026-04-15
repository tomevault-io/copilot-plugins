## datastackbuilder

> Data pipeline architecture tool. Design, simulate, and optimize data stacks

# DataStackBuilder

Data pipeline architecture tool. Design, simulate, and optimize data stacks
using 58+ real-world tools across AWS, Azure, GCP, and open-source ecosystems.

## Architecture

- `frontend/` — React 19 + Vite + Excalidraw canvas + Jotai state management
- `backend/` — FastAPI + NetworkX graph engine + physics-based simulation
- `_archive/` — Pre-restructure snapshot (do NOT modify or delete)

## Quick Start

```bash
# Backend (port 8000)
cd backend && pip install -r requirements.txt && uvicorn main:app --port 8000

# Frontend (port 5001)
cd frontend && npm install && npm run dev
```

## Frontend Structure (`frontend/src/`)

| Directory | Purpose |
|-----------|---------|
| `app/` | App shell (`App.tsx`), global providers |
| `stores/` | Jotai atoms — all global state lives here |
| `hooks/` | Custom hooks that compose atoms + side effects |
| `features/` | Feature modules organized by domain |
| `features/canvas/` | Excalidraw wrapper, overlays, node settings |
| `features/sidebar/` | Tool sidebar, ecosystem selector, tool buttons |
| `features/toolbar/` | Top toolbar, theme toggle, mode toggle |
| `features/simulation/` | Simulation panel, results display |
| `features/advisor/` | Stacky advisor panel |
| `features/modals/` | All modal dialogs (components, editor, source config, requirements) |
| `features/tool-info/` | Tool info panel |
| `shared/` | Reusable UI primitives (icons, base modal) |
| `services/` | API client — all backend fetch calls |
| `types/` | Shared TypeScript interfaces |
| `data/` | Static data: tool definitions (split by ecosystem), config schemas |
| `utils/` | Pure utility functions (layout, graph building, node rendering) |
| `styles/` | SCSS stylesheets |

## Backend Structure (`backend/`)

| Directory | Purpose |
|-----------|---------|
| `api/` | FastAPI route handlers (simulate, validate, tools) |
| `engine/` | Simulation orchestration, graph analysis, advisor logic |
| `kernels/` | Per-category simulation kernels (streaming, storage, etc.) |
| `data/profiles/` | JSON tool performance profiles (latency, throughput, cost) |
| `models/` | Pydantic request/response models |
| `tools/` | Tool metadata catalog |

## Key Concepts

- **Smart Nodes**: Excalidraw grouped elements with `customData.dataStackTool` representing a pipeline tool
- **Pipeline Graph**: Extracted from canvas arrows connecting smart nodes, sent to backend for simulation
- **Kernels**: Physics-based simulators per tool category (M/M/1 queueing, log-normal distributions)
- **Advisor (Stacky)**: Rule-based recommendation engine analyzing simulation results
- **Tool Profiles**: JSON configs defining latency, throughput, cost params per tool

## Conventions

- **Components**: PascalCase files, one component per file, aim for <300 lines
- **Hooks**: `use` prefix, one concern per hook
- **State**: Global state in `stores/` as Jotai atoms, local state with useState
- **Types**: Shared types in `types/`, component-specific types colocated
- **Backend**: snake_case files/functions, type hints required, Pydantic for all API models
- **Data files**: JSON for static config data, TS/Python only for logic
- **Imports**: Use relative paths from `src/` root

## Testing

```bash
# Frontend
cd frontend && npm run build   # Type-check + bundle

# Backend
cd backend && python -m pytest  # When tests exist
```

## Tool Categories

streaming, storage, warehouse, database, processing, orchestration, monitoring, ml, governance, quality

## Ecosystems

aws, azure, gcp, opensource

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shiv-aurora) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
