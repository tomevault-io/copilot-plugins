## pac-man-web-ai-generated

> 3D Pac-Man web game rendered as an office building using React Three Fiber. Each level is a floor (10 total), connected by elevator mechanic. Frontend SPA with a FastAPI leaderboard backend.

# 3D Pac-Man Web Game

## Project Overview
3D Pac-Man web game rendered as an office building using React Three Fiber. Each level is a floor (10 total), connected by elevator mechanic. Frontend SPA with a FastAPI leaderboard backend.

## Tech Stack
- **Build**: Vite 8.x
- **UI**: React 19.2 + TypeScript 5.9
- **3D**: React Three Fiber v9 + Drei v10
- **State**: Zustand v5
- **Serve**: nginx:alpine (Docker)
- **Deploy**: Docker + Traefik + GitHub Actions → pacman.enricd.com

## Commands

### Frontend
- `npm run dev` — Start dev server with HMR
- `npm run build` — TypeScript check + Vite build
- `npm run lint` — ESLint
- `npm run preview` — Preview production build

### Backend (run from `backend/`)
- `uv run fastapi dev app/main.py` — Start API dev server with hot reload
- `uv run pytest` — Run all backend tests (65 tests)
- `uv run pytest -v` — Run tests with verbose output
- `uv run pytest tests/test_submit_score.py` — Run a specific test file
- `uv lock` — Lock dependencies after changing `pyproject.toml`
- `uv sync --dev` — Install all dependencies including dev (pytest)

## Skill Files (`.claude/skills/`)

**Always check the relevant skill file before writing code that uses a library.** Skill files contain verified, up-to-date API references and patterns for each tool in the stack.

| Skill File | Covers |
|------------|--------|
| `react-three-fiber.md` | R3F v9 Canvas, useFrame, useThree, InstancedMesh, events + Drei v10 helpers |
| `zustand.md` | Zustand v5 create, selectors, getState, middleware, R3F integration pattern |
| `three-js.md` | Three.js 0.183 geometries, materials, lights, cameras, InstancedMesh, math |
| `vite-react-ts.md` | Vite 8 config/HMR/env + React 19 hooks/patterns + TypeScript 5.9 rules |
| `deployment.md` | Traefik labels, compose patterns, GitHub Actions CI/CD, nginx SPA config |
| `fastapi-backend.md` | FastAPI, SQLModel, SlowAPI, pydantic-settings, uv — verified API patterns |

### Rules for Using Skill Files
1. **Read the relevant skill file** before writing or modifying code that uses that library. Do not rely solely on training knowledge — the skill file has the verified, version-specific API.
2. **If a skill file doesn't cover what you need**, use `npx ctx7@latest` to fetch the latest docs (the lookup command is at the top of each skill file), then **update the skill file** with the new information.
3. **Keep skill files current**: When upgrading a dependency or discovering a new pattern, update the relevant skill file so future conversations benefit.
4. **Use the find-docs skill** (`npx ctx7@latest docs <libraryId> "<query>"`) when you encounter an API you're unsure about. Never guess at API signatures — look them up.

## Architecture Rules

1. **Pure game logic in `systems/`** — No React or Three.js imports. Just functions that take state and return new state.
2. **Single Zustand store** (`stores/gameStore.ts`) — No prop drilling, no React context for game state.
3. **InstancedMesh for repeated geometry** — Walls (`Wall.tsx`) and pellets (`Pellet.tsx`) use instancing.
4. **Constants in `utils/constants.ts`** — No magic numbers in components or systems.
5. **Grid-based movement** — All game logic operates on integer grid positions. 3D is rendering only.
6. **`gridToWorld()` for coordinate conversion** — Grid (row, col) → world (x, z). Y is up.
7. **Frame-rate independent** — All movement uses `delta` from `useFrame`.

## Project Structure
- `src/types/` — TypeScript type definitions
- `src/stores/` — Zustand store
- `src/hooks/` — React hooks (useKeyboard, useGameLoop)
- `src/systems/` — Pure game logic (movement, collision, ghostAI, mazeGenerator)
- `src/components/` — R3F 3D components (maze, characters, items, mechanics, camera)
- `src/ui/` — HTML overlay components (HUD, screens)
- `src/utils/` — Constants and helpers
- `src/api/` — Frontend API client (leaderboard)
- `.claude/skills/` — Up-to-date API reference docs for each library
- `backend/app/` — FastAPI leaderboard API (config, models, routes, database)
- `backend/tests/` — Backend pytest test suite

## Testing

### Backend Tests (`backend/tests/`)
Run: `cd backend && uv run pytest -v`

Uses **in-memory SQLite** (via `StaticPool`) and FastAPI's `dependency_overrides` for isolated tests. Rate limiting is disabled in most fixtures and tested separately.

| File | Covers | Count |
|------|--------|-------|
| `test_health.py` | Health endpoint returns 200 | 1 |
| `test_submit_score.py` | Valid submissions, all validation rules (username, score, level, duration), missing/invalid fields, type errors | 30 |
| `test_leaderboard.py` | Empty/populated leaderboard, sort order, tiebreaker, limit bounds, response shape, IP exclusion | 13 |
| `test_models.py` | Pydantic model validation independent of API (boundaries, unicode, emoji, stripping) | 13 |
| `test_rate_limiting.py` | Rate limit allows 5/hour, blocks 6th, doesn't affect GET endpoints | 4 |

**Rules:**
1. **Always run tests** after modifying backend code: `cd backend && uv run pytest`
2. **Add tests** for any new endpoint or validation rule
3. **Each test file** gets its own `conftest.py` fixture where needed
4. **Rate limit tests** use a separate fixture with `limiter.enabled = True` and `limiter.reset()`

## Maze Generation
- Recursive backtracking on left half, mirrored to right for symmetry
- Ghost house centered, with door above
- At least 2 horizontal teleport corridors
- Flood-fill validation ensures all paths reachable
- Odd levels: new maze. Even levels: same maze with invisible walls

## Ghost AI
| Ghost | Color | Chase Behavior |
|-------|-------|---------------|
| Blinky | Red | Targets Pac-Man's tile |
| Pinky | Pink | Targets 4 tiles ahead |
| Inky | Cyan | Flanking (Blinky + Pac-Man vector) |
| Clyde | Orange | Chase if far, scatter if close (<8 tiles) |

Modes: Scatter (7s) → Chase (20s) → repeat. Power pellet → Frightened.

## Deployment

Two-service deployment on Hetzner VPS behind Traefik:

| Service | Image | Dockerfile | Routing |
|---------|-------|-----------|---------|
| `web` | `ghcr.io/${REPO}:latest` | `./Dockerfile` (node → nginx) | `Host AND !PathPrefix(/api)` |
| `api` | `ghcr.io/${REPO}-api:latest` | `backend/Dockerfile` (uv + FastAPI) | `Host AND PathPrefix(/api)` |

- **Path-based routing** via Traefik labels — same origin, no CORS needed
- **SQLite persistence** via named volume `leaderboard-data:/app/data`
- **Rate limiting** at both Traefik (100/min) and application (5 submissions/hour) levels
- GitHub Actions **matrix build** builds both images in parallel → SSH deploy
- See `.claude/skills/deployment.md` for the full pattern and label conventions

## Tutorial Documentation (`docs/`)

This project doubles as a **learning exercise**. The user is an experienced backend/Python developer refreshing and learning frontend: TypeScript, React, React Three Fiber, Zustand, and Vite. They did some JS and React in the past but don't remember much.

### Writing Rules for `docs/`
1. **Every feature gets a tutorial doc** — not just "what was built" but "what you need to understand to build it yourself".
2. **Explain concepts from scratch** — assume the reader knows Python well and has vague JS/React memories. Draw parallels to Python/backend when useful (e.g. "TypeScript interfaces are like Python dataclasses/TypedDicts", "useEffect is like a context manager's `__enter__`/`__exit__`").
3. **Cover the full stack of concepts** needed for each feature:
   - **TypeScript fundamentals**: types, interfaces, generics, enums-as-objects, `as const`, union types, utility types — compared to Python's type hints.
   - **React fundamentals**: components, JSX, props, state (`useState`), effects (`useEffect`), refs (`useRef`), hooks, re-rendering — compared to server-side template rendering.
   - **React Three Fiber**: `<Canvas>`, `useFrame`, declarative 3D, meshes/geometries/materials, `InstancedMesh`, refs for imperative mutation.
   - **Zustand**: why not Redux/Context, creating stores, selectors, subscriptions — compared to Python global state patterns.
   - **Vite/npm**: modules, bundling, HMR, `package.json`, dev vs production — compared to pip/poetry.
   - **Deployment**: Docker multi-stage, nginx, Traefik, CI/CD — these the user likely knows, but document the frontend-specific parts.
4. **Use real code from this project** — every concept should point to the actual file and line where it's used. No abstract examples.
5. **Progressive complexity** — docs are numbered and build on each other. Earlier docs explain more basics, later ones assume prior docs were read.
6. **Include "Try it yourself" prompts** — small exercises or experiments the reader can do to verify understanding (e.g. "change the wall color in constants.ts and see it update live via HMR").

### Doc Index
See `docs/plan.md` for the full list and status of tutorial docs.

## Plan Tracking

The implementation plan lives at `docs/plan.md`. It contains all phases, tasks, and their status.

### Rules
1. **Check the plan** at the start of each conversation to understand current status.
2. **Update task status** in `docs/plan.md` as work progresses — mark items `[x]` when done, add new sub-items if needed.
3. **Update the Verification Checklist** at the bottom of the plan when a phase is confirmed working in-browser.
4. **Never change the plan structure or scope without asking the user first.** If a task turns out to need a different approach, or if new work is discovered, propose the change and wait for confirmation before modifying the plan.
5. **Keep the plan as the single source of truth** for what's done, in progress, and remaining.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enricd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
