## seed-viewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Getting Started

Before working on any task, read `README.md` first to understand the project structure. Then read the specific doc(s) relevant to your task:
- Frontend work → `docs/frontend.md`
- Backend/API work → `docs/backend.md`
- Model or animation work → `docs/models-and-animation.md`
- Deployment/Docker work → `docs/deployment.md`

## What This Is

A fullstack web app for browsing the BONES-SEED dataset. Renders skeleton animations on 3D character models (Three.js) and displays metadata from parquet files. Supports two character models: SOMA (default) and G1 robot. Deployed via Docker.

## Build & Run Commands

**Frontend** (from `frontend/`):
- `npm run dev` — Vite dev server on localhost:5173 (proxies `/api` to localhost:8080)
- `npm run build` — production build to `frontend/public/dist/`

**Backend** (from `backend/`):
- `pdm install` — install Python dependencies
- `DATA_ROOT=/path/to/dataset PORT=8080 python src/main.py` — runs FastAPI/uvicorn on port 8080

**Docker** (full stack):
- `./shdocker.sh` — builds and runs everything; edit `DATA_PATH` and `PORT` in the script first
- `DATA_PATH` points to root of the extracted dataset (containing `metadata/`, `files/`)
- Production port: 8666 (configurable)

**Lint**: `npx eslint` from `frontend/`

**Tests** (E2E with Playwright, from `frontend/`):
- `npx playwright test` — run all tests (headless Chromium)
- `npx playwright test tests/g1-model-switch.spec.ts` — run a single test file
- `npx playwright test --headed` — run with visible browser
- Tests expect the app running at `http://localhost:8666`

## Architecture

**Two-service architecture**: Vite frontend + FastAPI backend. In production, FastAPI serves the built frontend static files from `dist/`.

### Frontend (`frontend/public/src/`)

Vanilla TypeScript + Three.js, no framework. Single HTML entry point: `index.html`.

**State management**: `globals.ts` exports a single `g` object containing all shared state — Three.js objects (`RENDERER`, `SCENE`, `CAMERA`), animation state (`FRAME`, `PLAYING`, `LOOP_START/END`), the current `MODEL3D` instance, and `CURRENT_MODEL` type. There is no event bus; modules import `g` directly.

**Multi-model system**: `MODEL_CONFIGS` in `globals.ts` defines per-model settings (URL, scale, rotation, animation format, API endpoint). `ModelType = 'soma' | 'g1'`. The `model_selector.ts` handles switching — it disposes the current model, creates the right class (`Model3D` vs `Model3DG1`), loads its FBX, applies scene/lighting settings, reloads the same animation, and preserves UI state (playing, following, skeleton visibility).

**Model classes** (`models.ts`):
- `Model3D` — SOMA. Loads FBX, applies MeshStandardMaterial, uses `SkeletonHelper2`. Animation via BVH (quaternion tracks applied recursively from root bone). SOMA bones have underscore prefix (`_Hips`); `toBvhName()` strips it for BVH track matching.
- `Model3DG1` — G1 robot. Loads FBX with URDF-style dual hierarchy: converts SkinnedMeshes to regular Meshes parented under joints, removes `_pelvis_grp`. Uses `JointHelper` for skeleton viz. Animation via CSV (single-DOF revolute joints: pitch→Y, roll→X, yaw→Z).

**Animation system**:
- `animation.ts` — `Animation` class wrapping Three.js `BVHLoader` result. `loadBVHUrl()` fetches from backend, `loadBVHString()` loads from string.
- `g1_animation.ts` — `G1Animation` class parsing CSV with root position/rotation + per-joint DOF columns. `getFrameData(frame)` interpolates between frames.
- `__requestBVH.ts` in `browser/` dispatches to the right format based on `MODEL_CONFIGS[g.CURRENT_MODEL].animFormat`.

**Render loop** (`main.ts`): `animate()` → updates `FRAME` based on delta time → calls `MODEL3D.setFrame(FRAME)` → runs `UPDATE_LOOP` callbacks → updates controls (skipped during view cube animation) → renders scene.

**File browser** (`browser/` directory, ~11 modules): Table-based UI with pagination, search/filtering, row click loads animation. Entry point is `initBrowser()`.

### Backend (`backend/src/`)

- `globals.py` — loads metadata (.parquet) from `DATA_ROOT/metadata/`, sets up file paths at `DATA_ROOT/files/`, loads temporal labels from JSONL in metadata dir. Uses `filename` column as the DataFrame index. `DATA_ROOT` defaults to `/mnt/bones`.
- `storage_local/` routes:
  - `GET /api/storage_local/somabvh/?bvhpath=<filename>` — returns SOMA BVH file content (reads `move_soma_uniform_path`)
  - `GET /api/storage_local/g1csv/?csvpath=<filename>` — returns G1 CSV file content (reads `move_g1_path`)
  - `GET /api/storage_local/temporal_labels/?filename=<name>` — returns temporal label events for an animation
  - All animation endpoints support `?random=true` for random selection
- `metadata/` routes: `GET /api/metadata/<filename>` for single row, `GET /api/metadata/?page=&perpage=&query=` for paginated listing
- Expects Docker volumes: metadata at `/mnt/bones/metadata/` (parquet + temporal labels JSONL), data files at `/mnt/bones/files/` (containing `soma_uniform/bvh/`, `g1/csv/`, etc. — paths are relative as stored in metadata)

### Data Flow

User selects animation in browser → frontend fetches BVH/CSV via `/api/storage_local/somabvh/` or `/api/storage_local/g1csv/` based on `MODEL_CONFIGS[g.CURRENT_MODEL].animEndpoint` → parser creates `Animation` or `G1Animation` → assigned to `MODEL3D.anim` → render loop calls `setFrame()` each frame → Three.js renders. In parallel, `temporal_labels.ts` fetches temporal text events from `/api/storage_local/temporal_labels/` and displays them as a movie-style overlay synced to the current frame.

## Key Config

- Node 20.10.0 (`.nvmrc`), Python 3.11
- Tailwind custom color: `bones: #eafd4b`
- CORS allows localhost:5173 for dev
- `SharedArrayBufferMiddleware` adds cross-origin headers (needed for WASM loaders)

## Workflow

- Always deploy at the end of changes by running `./shdocker.sh` from the repo root.

## Lighting Settings

Per-model lighting settings system in `lighting_settings.ts` + `settings_panel.ts`:
- Each model type has independent defaults (hemisphere, key/fill/rim lights, env map, tone mapping, body/dark material params)
- Settings persisted to `localStorage` keyed by model type (`viewer_lighting_soma`, etc.)
- Version field for migration — bumping version forces reset to new defaults
- SOMA uses `MeshStandardMaterial`; G1 uses `MeshStandardMaterial`
- Settings panel (cog icon) provides real-time sliders; env map requires explicit "Regenerate" button

## Conventions

- Frontend browser modules use `__` prefix naming convention (e.g., `__requestBVH.ts`, `__localGlobals.ts`)
- G1 debug logging uses `[G1 FBX]`, `[G1 Animation]`, `[G1 setFrame]` prefixes
- SOMA FBX bone names have underscore prefix (`_Hips`, `_Spine1`); BVH track names are unprefixed (`Hips`, `Spine1`)
- G1 FBX joint names have leading underscore (e.g., `_left_hip_pitch_joint`); CSV column names don't (e.g., `left_hip_pitch_joint_dof`)
- Backend uses `loguru` for logging
- Backend endpoints follow `_<name>.py` naming in `storage_local/`, registered in `storage_local/main.py`
- Model rotation uses `'YXZ'` Euler order (avoids gimbal lock for G1's two-axis rotation)

---
> Source: [bones-studio/seed-viewer](https://github.com/bones-studio/seed-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
