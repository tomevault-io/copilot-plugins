## robot-link-forge

> Web-based tool for authoring/visualizing robot hand models (links, joints, tendons,

# Robot Link Forge

Web-based tool for authoring/visualizing robot hand models (links, joints, tendons,
sensors, obstacles), exporting to URDF / MJCF / Gazebo / Isaac, recording motion, and
**live-driving the robot from Manus data gloves** (the current focus of active work).

## Architecture

```
Manus SDK (ROS2)  ─/manus_glove_{0,1}→  glove_bridge.py  ─WS 30Hz→  frontend store
   hardware            (ROS2 node)        (ws://:8765)         (zustand) → 3D view

Backend (FastAPI, :8000)  ←REST→  Frontend (React+Vite+three.js, :5173)
   exporters/, saved_*/                 src/frontend/src/
```

- **Frontend**: React + TypeScript + Vite, `three.js` via `@react-three/fiber`/`drei`,
  state in **zustand** (`src/frontend/src/store.ts` — large, single store). MediaPipe for
  camera hand tracking. Tailwind for styling.
- **Backend**: FastAPI (`src/backend/main.py`). Pydantic models in `robot_models.py`.
  Exporters in `src/backend/exporters/`. Persists to `saved_projects/`,
  `saved_recordings/`, `saved_glove_mappings/` (JSON / zip).
- **Glove bridge**: `src/backend/glove_bridge.py` subscribes to Manus ROS2 topics and
  rebroadcasts ergonomics data over a WebSocket. Frontend connects same-origin via the
  Vite proxy path `/glove-ws` (works over HTTPS / LAN IP). See `run_glove_bridge.sh`.

## Run

```bash
bash run_all.sh                 # backend + frontend (+ glove bridge); --no-glove to skip
# or individually:
bash src/backend/run_backend.sh         # FastAPI on :8000
cd src/frontend && npm run dev -- --host # Vite on :5173
bash src/backend/run_glove_bridge.sh    # needs ROS2 + Manus dashboard/hardware
```

There is no test/lint script wired up in `package.json` (only `dev`/`build`/`preview`).
Backend has pytest tests under `src/backend/tests/`.

## Glove retargeting (active work)

Data flow: `connectGlove()` opens the WS; a 30Hz `setInterval` in the store reads
`gloveMappings` and writes joint/tendon values. Key pieces:
- Types: `GloveMapping`, `GloveData`, `GloveMappingPreset`, `GloveFingerSource` in `types.ts`.
- Sync loop + preset actions: `store.ts` (search `connectGlove`, `addGloveMapping`,
  `saveGloveMappingPreset`).
- UI: `src/frontend/src/components/GlovePanel.tsx` (connection, live data, mapping table,
  preset manager).
- Source channels = 5 fingers × {Spread, MCP/PIP/DIP Stretch} ergonomics from the glove.
- Retarget math: linear map `[inputMin,inputMax]→[0,1]→[targetMin,targetMax]` (deg/mm),
  clamped to joint limits; DOF-disabled axes are skipped; optional collision bypass.

### Robot identity caveat
Scene JSON (`robot-scene.json` inside project zips) has **no robot name/id** at the top
level, and joint `targetId`s are per-import UUIDs. So glove mappings/profiles must
re-bind to joints **by name**, and robots loaded by filename should get a name injected
(retrofit old data or auto-fill on load) — see the project memory for the plan.

## Conventions
- Angles stored in **radians** internally; UI shows **degrees** (joints) / **mm** (prismatic).
- Commit messages: conventional-commit style (`feat(scope):`, `fix(glove):`, …).

---
> Source: [younlea/robot-link-forge](https://github.com/younlea/robot-link-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
