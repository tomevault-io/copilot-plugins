## terrascry

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TERRASCRY (*terra* + *scry* — "seeing what the earth hides") is a unified multi-sensor geophysical survey platform combining three instruments in a single monorepo.

| Project | Path | Purpose |
|---------|------|---------|
| Pathfinder | `pathfinder/` | Handheld multi-sensor fluxgate gradiometer for rapid reconnaissance |
| HIRT | `hirt/` | Crosshole dual-channel subsurface tomography (MIT-3D + ERT-Lite) |
| GeoSim | `geosim/` | Physics simulation engine for both instruments |
| Shared | `shared/` | Cross-instrument firmware, protocols, hardware, and deployment |

Each sub-project has its own `CLAUDE.md` with project-specific conventions:
- `pathfinder/CLAUDE.md` — Firmware commands, multi-sensor architecture, design principles
- `hirt/CLAUDE.md` — Quarto docs, writing style (narrative prose for theory sections)
- `geosim/CLAUDE.md` — Physics standards, testing conventions, code style

## Commands

### Pathfinder (ESP32 / PlatformIO)
```bash
cd pathfinder/firmware && pio run -e esp32dev           # Build ESP32 firmware
cd pathfinder/firmware && pio run -e esp32dev -t upload  # Upload to ESP32
cd pathfinder/firmware && pio run -e nanoatmega328       # Build legacy Nano firmware
cd pathfinder/firmware && pio test -e native             # Run native (host) unit tests
```

PlatformIO environments: `esp32dev` (main), `nanoatmega328` (legacy handheld), `nano_drone` (drone variant), `native` (host-side unit tests, no hardware needed).

### HIRT (Quarto documentation)
```bash
cd hirt/docs && quarto render              # Full render (HTML + PDF)
cd hirt/docs && quarto preview             # Live reload preview
cd hirt/docs && quarto render --to html    # HTML only
cd hirt/docs && quarto render --to pdf     # PDF only
```

### GeoSim (Python)
```bash
cd geosim && pip install -e ".[dev]"       # Install for development
cd geosim && pip install -e ".[all]"       # Install everything (viz + hirt + dev)
cd geosim && pytest tests/                 # Run all tests
cd geosim && pytest tests/ -k dipole       # Run tests matching keyword
cd geosim && pytest tests/test_zmq_server.py               # Single test file
cd geosim && pytest tests/test_zmq_server.py::test_ping    # Single test function
cd geosim && ruff check geosim/            # Lint
cd geosim && geosim-server --scenario scenarios/single-ferrous-target.json  # Start ZeroMQ server
```

**Optional dependency groups:** `dev` (pytest, ruff, pandas), `viz` (PyVista, matplotlib, Jupyter), `hirt` (SimPEG, pyGIMLi), `all` (everything).

**Pytest markers:** `simpeg` (requires SimPEG), `pygimli` (requires pyGIMLi), `slow`. Deselect heavy deps with `-m "not simpeg"`.

**Ruff config:** line-length=100, target Python 3.10+, rules: E, F, W, I, N, UP.

### Data Visualization
```bash
python pathfinder/firmware/tools/visualize_data.py <file.csv>        # Time series
python pathfinder/firmware/tools/visualize_data.py <file.csv> --map  # Spatial map
```

## CI/CD

GitHub Actions workflows are **path-filtered** — each sub-project's CI only triggers on changes to its own directory plus `shared/`:

| Workflow | Trigger Paths | Jobs |
|----------|--------------|------|
| `geosim-ci.yml` | `geosim/**`, `shared/**` | pytest + ruff lint |
| `pathfinder-ci.yml` | `pathfinder/**`, `shared/**` | PlatformIO build + native tests |
| `hirt-ci.yml` | `hirt/**`, `shared/**` | Quarto render (HTML) + lint |
| `shared-ci.yml` | `shared/**` | Header validation + protocol docs + Docker Compose |

Changes to `shared/` trigger CI for **all** sub-projects.

## Architecture

### Cross-Project Data Flow
```
Pathfinder (surface survey, 10 Hz)
     │ MQTT over WiFi
     ▼
NVIDIA Jetson (edge compute)  ◄── HIRT (crosshole, per-sequence)
     │ tilt correction, anomaly detection, data logging
     ▼
Workstation / HPC
     │ SimPEG joint inversion (magnetics + EMI + MIT + ERT)
     ▼
3D multi-physics subsurface model
```

### GeoSim Data Flow
Scenario JSON → physics engine → sensor data → CSV/visualization/ZMQ

Core module: `geosim/magnetics/dipole.py` (dipole field, superposition, gradiometer readings — NumPy only, no viz deps). The ZeroMQ server (`geosim/server.py`) wraps physics for external clients (Godot frontend in `godot/`). Streaming module (`geosim/streaming/`) publishes via MQTT.

### Shared Sensor Pod
Shared hardware between Pathfinder and HIRT: ZED-F9P RTK GPS (0x42), BNO055 IMU (0x29), BMP390 barometer (0x77), DS3231 RTC (0x68) in an IP67 enclosure. Connected via M8 8-pin connector and PCA9615 differential I2C over Cat5 STP cable (1-2m).

Both instruments use the same GPS receiver — eliminates coordinate transformation errors and enables seamless data fusion in joint inversion.

**Firmware integration:** Both PlatformIO projects reference shared libs via `lib_extra_dirs = ../../shared/firmware` in `platformio.ini`.

### MQTT Streaming
All instruments stream through MQTT to the Jetson edge node. Topic hierarchy:
```
terrascry/{instrument}/data/raw         — Raw sensor readings
terrascry/{instrument}/data/corrected   — After tilt/orientation correction
terrascry/{instrument}/anomaly/detected — Real-time anomaly flags
terrascry/{instrument}/model/update     — Latest model output
terrascry/status/{instrument}           — System health (1 Hz)
```
QoS levels: 0 for raw data (loss tolerable), 1 for corrected data (logged), 2 for anomaly alerts (critical). Messages are JSON with ISO 8601 UTC timestamps. Full spec in `shared/protocols/mqtt_topics.md`.

### Joint Inversion
Pathfinder surface data serves as boundary conditions for HIRT 3D inversion:
- Magnetic gradient map constrains top susceptibility layer
- EMI conductivity constrains top resistivity layer
- LiDAR DEM corrects inversion mesh for terrain
- Cross-gradient regularization (Gallardo & Meju 2003) couples multi-physics models in SimPEG

See `geosim/docs/research/joint-inversion-concept.md`.

### Edge Deployment
NVIDIA Jetson AGX Orin 64GB running Docker containers: Mosquitto MQTT broker, per-instrument processing pipelines, cloud bridge via Starlink. Config in `shared/edge/`.

## Platform Conventions

### Performance Claims
All specifications across all sub-projects must use these qualifiers:
- **(Measured)** — Bench or field tested
- **(Modeled)** — Theoretical analysis or simulation
- **(Target)** — Design goal, not yet validated

### Units
SI units throughout. Right-handed coordinate system: X=East, Y=North, Z=Up. Positions in meters, magnetic fields in Tesla (T), gradients in T/m or nT, angles in degrees.

### Voice
Active voice, first person plural for design decisions ("We selected..." not "It was decided...").

### Consensus Validation
Critical design decisions are validated using multi-model consensus via PAL MCP `consensus` tool:
- **Models:** `google/gemini-3.1-pro-preview` + `openai/gpt-5.3-codex` (OpenRouter namespace format, neutral stance)
- Issues labeled `needs-planning` require consensus validation before implementation

## Anti-Drift System

This project uses a Contract & Guard system to prevent goal drift across sessions.

### Session Rituals
- **Start of session or task switch:** Run `/orient` (or its steps manually) before editing files.
- **Task complete:** Run `/done` to verify acceptance and stop.
- **Ending session:** `/handoff` auto-runs via Stop hook, or run manually for richer context.
- **Starting new epic:** Run `/start-epic <name>` to reset scope.
- **Mid-session check:** Run `/checkpoint` at natural transition points.

### Scope Enforcement
- `.claude/current-focus.md` defines `allowed_paths`, `acceptance`, and `stop_when`.
- A PreToolUse hook blocks edits outside `allowed_paths`. To edit outside scope, update `current-focus.md` first.
- When `stop_when` is satisfied, **STOP**. Do not add cleanup, docs, comments, refactors, or extra tests unless explicitly listed in `acceptance`.

### Memory Rules
- Never use memory to store task progress or volatile state — that belongs in `.claude/current-focus.md`.
- Memory is for durable facts: architecture decisions, user preferences, deployment gotchas.
- `.claude/handoff.md` handles inter-session continuity (auto-written on session end).

## Workflow

This project follows the AI-Assisted Development Harness (see `brain/ai-assisted-development-harness.md`).

### Development Loop
```
Scope → Deliberate (if non-trivial) → Contract → Build → Evaluate → Review → Ship
```

Three roles, separated by design:
| Role | Who | Rule |
|------|-----|------|
| **Planner** | AI Counsel (`mcp__ai-counsel__deliberate`) | Decides *what* to build and *how* |
| **Builder** | Claude Code (primary session) | Implements against acceptance criteria |
| **Evaluator** | `.claude/agents/evaluator.md` (independent agent) | Judges the running product, not the code |

The builder never evaluates its own work. The evaluator never fixes what it finds.

### Sprint Contracts
`.claude/current-focus.md` is the sprint contract. It defines `acceptance` (observable behaviors), `stop_when` (completion criteria), and `allowed_paths` (scope guard). Write acceptance criteria before coding. The evaluator reads this file to know what to test.

### Key Rules
- Gitflow branching: `main` → `develop` → `feature/*`. Never commit directly to `main` or `develop`.
- Issues labeled `needs-planning` require AI Counsel deliberation before implementation.
- AI Counsel models: `gpt-5.4` (Codex CLI) + `gemini-3.1-pro-preview` (Gemini CLI)
- Run tests after every edit (PostToolUse hook on Edit|Write).
- One commit per epic, referencing closed issues (`closes #N`).
- Push to `develop` only. PRs for merging to `main`.

### Autopilot
- Do autonomously: ceremony, boilerplate, tests, lint, config, git ops, straightforward issues
- Ask the user: epic-level planning approval, architecture decisions, scope changes, new service adoption
- Never ask: style, commit messages, branch names, formatting choices

### Testing
- TDD: write failing tests first, then implement
- Unit tests on every commit, integration on PR, E2E on merge to develop
- Meaningful coverage of business logic, not vanity metrics
- **Before committing:** run the evaluator agent against `current-focus.md` acceptance criteria

### AI Review
- **Evaluator agent** (`.claude/agents/evaluator.md`): exercises running app against acceptance criteria. Run before every commit.
- **Reviewer agent** (`.claude/agents/reviewer.md`): sends diff to AI Counsel for multi-model code review. Run for non-trivial changes.
- Self-review diffs before committing
- AI Counsel conference mode for non-trivial PRs
- Subagents for parallel review passes (correctness, performance, security)

### When to Deliberate vs Just Build
| Situation | Action |
|-----------|--------|
| Straightforward implementation, clear path | Skip deliberation, go straight to build |
| Multiple valid approaches, architecture decision | AI Counsel conference mode (2 rounds) |
| Novel idea, unexplored territory | AI Counsel conference mode (3 rounds, add third model) |
| Bug fix with obvious cause | Skip deliberation, skip contract, just fix it |
| Refactor touching shared interfaces | Quick deliberation on approach, then build |

## 3D Scene Ownership

Both frontend (`SceneView.tsx`) and viewer (`ViewerScenePane.tsx`) use `Scene3DCore` and `SceneInspector` from `shared-ui/src/components/scene3d/`. When changing any of these shared concerns, modify in `shared-ui` first:

- Camera presets, controls, scaling
- Volume rendering (RayMarchVolume3D, CameraClipPlane)
- Clip plane, depth labels, scale markers
- Buried objects, Euler markers, subsurface pins
- Contour lines, placement flash
- Inspector panel sections (View, Surface, Volume, Slice, Pins)

App-specific content goes in the adapter via slots (`terrainSlot`, `clippedSlot`, `overlaySlot`, `appSections`). `shared-ui/` must never import from `frontend/` or `viewer/`.

## Licenses
- **Software/Firmware:** MIT License (`LICENSE-SOFTWARE.md`)
- **Hardware Designs:** CERN-OHL-S v2.0 (`LICENSE-HARDWARE.md`)

## Key Research Archives
- Pathfinder multi-sensor architecture: `pathfinder/research/multi-sensor-architecture/` (26 files)
- Interference matrix: `pathfinder/research/multi-sensor-architecture/interference-matrix.md`
- HIRT sensor pod consensus: `hirt/research/electronics/sensor-pod-consensus.md`
- Joint inversion concept: `geosim/docs/research/joint-inversion-concept.md`
- Streaming architecture: `geosim/docs/streaming-architecture.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chadlindell)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chadlindell)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
