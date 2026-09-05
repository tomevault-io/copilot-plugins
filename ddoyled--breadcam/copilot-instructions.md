## breadcam

> validates that synthetic sunrise produces expected monotonically increasing brightness.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Video stream capture and analysis pipeline of an RTSP stream. Captures streams via OpenCV, stores timelapsed frames in HDF5, and performs image analysis to detect brightness changes for tracking sourdough starter activity across feedings. Supports **cultures** — named groups of sessions for cross-feeding analysis.

## Project Structure

```
src/breadcam/
├── config.py           # dataclass-based settings, loads .env
├── capture.py          # RTSP single-frame grab via OpenCV
├── processing.py       # single-frame transformations (grayscale, blur, normalize, diff)
├── analysis.py         # batch frame analysis against anchor frames
├── storage.py          # HDF5 read/write (schema v2.0)
├── session.py          # session lifecycle orchestration
├── scheduler.py        # periodic capture loop
├── visualization.py    # GIF animation generation
├── synthetic_frames.py # shared synthetic sunrise utilities (testing/synthetic server)
├── commands.py         # programmatic API for all CLI operations
├── cli.py              # CLI entry point (breadcam command)
├── camera_store.py     # thread-safe camera registry (data/cameras.json)
├── culture_store.py    # thread-safe culture registry (data/cultures.json)
├── culture_analysis.py # cross-session analysis: SessionSummary + CultureAnalysis
├── utils.py            # session ID generation
├── time_parsing.py     # human-readable duration/interval parsing for CLI
└── api/
    ├── main.py         # FastAPI app factory, lifespan, CORS, static mounts
    ├── config.py       # APISettings dataclass (host, port, log_level, etc.)
    ├── models.py       # Pydantic request/response schemas, NaN-handling validators
    ├── dependencies.py # FastAPI DI with LRU caching (settings, camera store, culture store, task manager)
    ├── tasks.py        # background task lifecycle (TaskManager, CaptureTask, TaskStatus)
    ├── file_manager.py # temp file storage and auto-cleanup (gifs, images, sessions)
    ├── static/         # control.html/js/css, dashboard.html/js/css
    └── routers/
        ├── health.py   # GET /api/v1/health, GET /api/v1/config
        ├── grab.py     # POST /api/v1/camera/grab
        ├── cameras.py  # CRUD /cameras + POST /cameras/{id}/activate
        ├── captures.py # POST/GET/POST /captures (start, status, cancel); culture_id auto-assign
        ├── sessions.py # GET/POST /sessions (list filterable by culture, info w/ culture_id, analyze, file, analysis, frames)
        ├── cultures.py # CRUD /cultures + session assign/unassign + GET /cultures/{id}/analysis
        ├── control.py  # GET /control (camera management + culture management UI)
        ├── dashboard.py# GET /dashboard (analytics UI with culture selector)
        └── files.py    # GET /files/{category}/{filename}
```

## Commands

### Install dependencies
```bash
uv sync --all-extras
```

### Grab a single frame
**Requires synthetic RTSP server running** (see synthetic RTSP Server section).
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam grab -o frame.png
```

### Run a capture session
**Requires synthetic RTSP server running** (see synthetic RTSP Server section).
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 1.0 --interval 15
```

### Run a capture session with raw frame storage
**Requires synthetic RTSP server running** (see synthetic RTSP Server section).
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 1.0 --interval 15 --store-raw
```

**Note:** Enabling `--store-raw` stores both preprocessed frames and full-resolution raw BGR frames in the HDF5 session file. This increases file size by approximately 50-100% but enables:
- Debugging of the preprocessing pipeline (compare raw vs processed)
- Alternative analysis workflows requiring original color information
- ML/CV experiments needing full-resolution color data
- Post-capture reprocessing with different parameters

Use `--store-raw` only when raw frames are needed for specific analysis or debugging purposes.

### View session info
```bash
uv run breadcam info data/sessions/<session>.h5
```

### Generate visualization
```bash
uv run breadcam analyze data/sessions/<session>.h5 -o output.gif
```

### Generate visualization with custom anchor frame
```bash
uv run breadcam analyze data/sessions/<session>.h5 -o output.gif --anchor-frame 2
```

### Run tests
```bash
# Run all tests (including integration tests)
uv run pytest

# Run only integration tests
uv run pytest -m integration

# Skip integration tests (faster for development)
uv run pytest -m "not integration"

# Skip slow tests
uv run pytest -m "not slow"

# Run specific test file
uv run pytest tests/test_visualization_integration.py -v
```

### Run linting/formatting
```bash
uv run ruff check src/ tests/
uv run ruff format src/ tests/
```

### Run notebooks
```bash
uv sync --all-extras
uv run jupyter notebook
```

## Data Environment Management

The project supports environment-based data directory separation via the `BREADCAM_DATA_ENV` environment variable. This allows you to keep production captures separate from development/test data.

### Environments

Three environments are supported:
- **prod** (default): Production captures for long-running monitoring (e.g., sunrise tracking)
- **dev**: Development test captures during feature development. Not that for most purposes, this is the environment you should use.
- **test**: Experimental/throwaway data that can be safely deleted

### Directory Structure

Session data is stored in environment-specific subdirectories:
```
data/
├── prod/sessions/    # Production data (BREADCAM_DATA_ENV=prod)
├── dev/sessions/     # Development data (BREADCAM_DATA_ENV=dev)
└── test/sessions/    # Test data (BREADCAM_DATA_ENV=test)
```

### Usage

Set the `BREADCAM_DATA_ENV` environment variable before running commands:

```bash
# Development workflow - quick test captures
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 0.1 --interval 1

# Production workflow - long sunrise monitoring
export BREADCAM_DATA_ENV=prod
uv run breadcam capture --duration 8.0 --interval 15

# Test workflow - experimental captures
export BREADCAM_DATA_ENV=test
uv run breadcam capture --duration 0.5 --interval 5
```

### Cross-Environment Analysis

The `info` and `analyze` commands accept explicit paths, so you can analyze sessions from any environment:

```bash
# Analyze a dev session while in prod environment
export BREADCAM_DATA_ENV=prod
uv run breadcam info data/dev/sessions/20260210_140000_abcd1234.h5
uv run breadcam analyze data/dev/sessions/20260210_140000_abcd1234.h5 -o dev_viz.gif

# View sessions from specific environment
ls data/dev/sessions/
ls data/prod/sessions/
```

### Default Behavior

If `BREADCAM_DATA_ENV` is not set, the system defaults to `prod` for backward compatibility. You can also set it in your `.env` file (see `.env.example`).

### Workflow Examples

**Development iteration cycle:**
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 0.01 --interval 0.5  # Quick test
DEV_SESSION=$(ls -t data/dev/sessions/*.h5 | head -1)
uv run breadcam analyze "$DEV_SESSION" -o test.gif
# Clean up when done: rm -rf data/dev
```

**Production monitoring:**
```bash
export BREADCAM_DATA_ENV=prod
uv run breadcam capture --duration 8.0 --interval 15  # 8-hour sunrise capture
PROD_SESSION=$(ls -t data/prod/sessions/*.h5 | head -1)
uv run breadcam analyze "$PROD_SESSION" -o sunrise.gif
```

## Key Dependencies

- **opencv-python** (`cv2`) — RTSP capture and image processing
- **numpy** — Array operations for image manipulation
- **h5py** — HDF5 session storage
- **python-dotenv** — Environment variable loading
- **matplotlib** — Visualization and GIF animation output (PillowWriter)
- **pillow** — Image I/O
- **fastapi** / **uvicorn** — REST API server
- **pydantic** / **pydantic-settings** — API request/response validation, API config

Python 3.10+, managed by uv. Virtualenv at `.venv/`.

## Architecture Notes

- **Config**: All settings via frozen dataclasses in `config.py`. Camera credentials loaded from `data/cameras.json` via `CameraStore` (managed via `breadcam-api` at `/control`). `BREADCAM_DATA_ENV` env var (or `.env`) controls data directory. Other modules receive config as constructor/function args — never read env vars directly.
- **Capture**: OpenCV-based RTSP frame grab with configurable retry. Connects, grabs one frame, disconnects each time.
- **Processing**: Single-frame transformations: grayscale → 25×25 box blur → normalize to [0,1]. Individual diff computation (frame vs anchor) with relative noise threshold.
- **Analysis** (schema v2.0): Batch frame analysis performed on-demand at visualization time. Computes diffs and metrics against a user-specified anchor frame. Anchor can be changed without re-capturing.
- **Storage** (schema v2.0): One HDF5 file per session under `data/{env}/sessions/`. Contains ONLY preprocessed frames with timestamps. No diffs or metrics stored. Gzip compressed. ~33MB for 50 frames (1920×1080). Optional `/raw_frames` group when `--store-raw` used.
- **Migration**: Schema v2.0 is a BREAKING CHANGE. Old session files (v1.0 with pre-computed diffs/metrics) are incompatible. Re-capture sessions with new code.
- **Session IDs**: Format `YYYYMMDD_HHMMSS_<8-char-hex>`
- **Camera Store**: Thread-safe registry in `camera_store.py`, backed by `data/cameras.json`. Manages add/update/delete/activate. First camera auto-activated. Managed via API at `/control` or `/cameras`.
- **Culture Store**: Thread-safe registry in `culture_store.py`, backed by `data/cultures.json`. Culture IDs format `cul_<8-char-hex>`. A session belongs to at most one culture (enforced by `add_session_to_culture` which removes prior membership). Managed via API at `/cultures` or `/control` Culture tab.
- **Culture Analysis**: `culture_analysis.py` computes per-session `SessionSummary` (peak brightness diff, time-to-peak, duration) and aggregates into `CultureAnalysis`. Invoked on-demand via `GET /cultures/{id}/analysis`. Sessions with missing HDF5 files or <2 frames are skipped.
- **API**: FastAPI app in `api/`. App factory pattern (`create_app()`). Background captures run in threads via `TaskManager`. Temp files (GIFs, session downloads) managed by `FileManager` with auto-cleanup. Session cache in sessions router (5-min TTL). Blocking I/O wrapped with `asyncio.to_thread()`.
- **API Endpoints**:
  - `GET /api/v1/health`, `GET /api/v1/config`
  - `GET|POST|PUT|DELETE /cameras`, `GET /cameras/active`, `POST /cameras/{id}/activate`
  - `POST /api/v1/camera/grab` — single frame PNG
  - `POST /captures`, `GET /captures/{id}/status`, `POST /captures/{id}/cancel`
  - `GET /sessions[?culture_id=&uncategorized=]`, `GET /sessions/{id}` (includes `culture_id`), `POST /sessions/{id}/analyze`, `GET /sessions/{id}/file`, `GET /sessions/{id}/analysis`, `GET /sessions/{id}/frames/{index}`
  - `GET|POST /cultures`, `GET|PUT|DELETE /cultures/{id}`, `POST /cultures/{id}/sessions/{session_id}`, `DELETE /cultures/{id}/sessions/{session_id}`, `GET /cultures/{id}/analysis`
  - `GET /files/{category}/{filename}`
  - `GET /control` (camera + culture management UI), `GET /dashboard` (analytics UI with culture filter)
- **Old POC files**: Archived in `archive/` (git-ignored, not committed)
- Credentials stored in `.env` — do not commit to public repositories

## Testing

### Test Categories

- **Unit tests**: Fast, isolated component tests (test_storage.py, test_analysis.py, test_culture_store.py, test_culture_analysis.py, etc.)
- **Integration tests** (`@pytest.mark.integration`): End-to-end pipeline tests (test_cultures_api.py, test_session_culture_api.py, test_capture_culture_autoassign.py)
- **Slow tests** (`@pytest.mark.slow`): Performance and large data tests

### Visualization Integration Test

The `test_visualization_integration.py` module validates the complete visualization pipeline
using synthetic sunrise data generated from a 2D Gaussian mathematical model.

**What it tests**:
1. Synthetic frame generation (deterministic, reproducible)
2. HDF5 session storage (schema v2.0)
3. Session loading via commands API
4. Frame analysis against anchor frame
5. GIF visualization generation
6. Output validation (format, frame count, dimensions)

**Purpose**: Catch regressions in visualization flow without manual inspection.

**Run time**: ~2-4 seconds per test (matplotlib GIF generation is slow).

**Validation approach**: Tests GIF metadata (format, frame count, dimensions) and
validates that synthetic sunrise produces expected monotonically increasing brightness.

### Synthetic Culture Data

`scripts/generate_synthetic_culture.py` creates a multi-session culture with synthetic HDF5 session files for testing culture analysis without live capture:

```bash
BREADCAM_DATA_ENV=dev uv run python scripts/generate_synthetic_culture.py
```

Generates 5 sessions with synthetic brightness curves, registers them in a new culture, and prints the culture ID.

### synthetic RTSP Server

`synthetic_rtsp_server.py` provides synthetic sunrise stream for testing without live camera.

**Requirements:**
- MediaMTX binary (`brew install mediamtx`)
- FFmpeg binary (`brew install ffmpeg`)

**Start server:**
```bash
uv run python synthetic_rtsp_server.py  # Port 8554, 10fps, 30min cycle
uv run python synthetic_rtsp_server.py --port 8554 --fps 30 --rise-duration 60
uv run python synthetic_rtsp_server.py --width 1920 --height 1080 --verbose
```

**Note:** MediaMTX and FFmpeg are auto-started/stopped by the script. No manual setup required.

**URL:** `rtsp://localhost:8554/stream`

**Integration test workflow:**
```bash
# Start synthetic RTSP server first (in a separate terminal)
uv run python synthetic_rtsp_server.py

# Add synthetic RTSP camera via control center (one-time setup):
#   http://localhost:8000/control → add camera with host=localhost, port=8554, stream_path=stream

# Capture test data
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 0.1 --interval 1

# Verify capture
DEV_SESSION=$(ls -t data/dev/sessions/*.h5 | head -1)
uv run breadcam info "$DEV_SESSION"
uv run breadcam analyze "$DEV_SESSION" -o test.gif
```

**Mathematical model:** Same 2D Gaussian as `test_visualization_integration.py`
- Sun position: `y = height * (0.8 - 0.4 * progress)`
- Brightness: `0.1 + 0.7 * progress * exp(-distance²/2σ²)`
- Loops at 100% progress (sun reaches 80% completion)

**Implementation:** MediaMTX RTSP server with FFmpeg publishing synthetic frames via stdin pipe. Both subprocesses auto-managed by script. Shared frame generation utilities in `src/breadcam/synthetic_frames.py`.

### Live Stream Tests

Any test or command that involves `grab` or `capture` requires the synthetic RTSP server to be running first. Start it in a separate terminal before running those tests:

```bash
uv run python synthetic_rtsp_server.py
```

Set `BREADCAM_DATA_ENV=dev` for all development and testing workflows. Reserve `prod` only for production deployments.

### When to Run Integration Tests

**Always run integration tests when**:
- Modifying `visualization.py`, `analysis.py`, or `storage.py`
- Changing HDF5 schema
- Updating commands API
- Modifying `culture_store.py`, `culture_analysis.py`, or cultures router
- Before creating pull requests
- Before releases

**May skip during**:
- Rapid iteration on unrelated features (use `-m "not integration"`)
- Quick unit test runs during development

## General Advice

- In all interactions and commit messages, be extremely concise and sacrifice grammar for the sake of concision.

## Plans

- At the end of each plan, give me a list of unresolved questions to answer, if any. Make the questions extremely concise. Sacrifice grammar for the sake of concision.

- When executing multiphase plans, write the plan to a file to persist plan execution status through multiple chats. Include at the end of the plan a checklist of tasks accomplished and a plan status update.

## DevOps

This repository uses the following conventions for branch names:
- main - production
- release/vXX.XX.XX - release branch to stage multiple feature branches. Follow semantic versioning
- hotfix/<hotfix-description> - hotfix branches that are intended to merge directly to main
- feature/<feature-description> - feature branches

Issues are tracked in Github to capture bug reports and feature requests. When interacting with Github issues, use the `gh` cli tool and the `issue` subcommand, eg `gh issue view` and `gh issue create`. 

When working on a new feature or bugfix, create a new branch in a git worktree following our standard naming conventions. Merge into the nearest release branch when you're done. If a release branch doesn't exist, make one to merge into. Follow semantic versioning. When you're done working on the issue, open a pull request into the release branch and close the issue

---
> Source: [ddoyled/breadcam](https://github.com/ddoyled/breadcam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
