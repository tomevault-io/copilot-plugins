## ghost-creator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Ghost Creator AI is a Windows desktop app that generates documentary-style short/long videos from a topic (research → script → critic → review → voice → footage → assembly → YouTube upload). It ships as an **Electron + React** GUI shell driving a **Python FastAPI sidecar**, with the pipeline itself implemented as a **stateful LangGraph** graph checkpointed to SQLite.

## Architecture: two processes, one machine

The Electron main process (`electron/`) spawns the Python API as a child process and talks to it over HTTP/WebSocket on `127.0.0.1:8766`.

- **`electron/python-bridge.ts`** resolves which Python to run, in order: `venv/Scripts/python.exe -m api.server` (dev) → `resources/GhostCreatorAPI/GhostCreatorAPI.exe` (packaged onedir) → `python -m api.server` (fallback). It health-checks `/health` and **reuses an already-running server** instead of erroring, so a manually started `python -m api.server` will be adopted by the GUI.
- **`api/server.py`** is the FastAPI app. Its `lifespan` eagerly compiles the LangGraph pipeline, bootstraps FFmpeg, and binds the progress broadcaster to the event loop. Routers live in `api/routes/` (`pipeline`, `config`, `history`, `upload`, `workshop`, `system`, `docs`, `misc`).
- **`src/`** is the Vite/React UI. Tabs in `src/tabs/` (`DocumentaryTab`, `EditorTab`, `HistoryTab`, `DirectUploadTab`, `SettingsTab`) map to the GUI sections.

### Progress: WebSocket with HTTP-polling fallback (important)
On Windows, `localhost` may resolve to IPv6 `::1` while uvicorn binds IPv4 `127.0.0.1`, breaking WebSockets. The system is dual-channel: `api/services/progress_broadcaster.py` keeps a monotonic sequence-numbered (`_seq`) sliding history of events; the React hook `src/hooks/usePipelineWebSocket.ts` uses the WS feed but **falls back to polling `/api/pipeline/progress?after=N`** if the socket drops. Sequence numbers dedupe across both channels. When adding pipeline events, emit through the broadcaster — don't bypass it.

## The LangGraph pipeline (`graph/`)

`graph/pipeline.py` wires all nodes and compiles the graph with a `SqliteSaver` checkpointer (`get_pipeline()` is a process-wide singleton). Nodes live in `graph/nodes/`; the shared state is a `TypedDict` in `graph/state.py`.

Flow: `research → script → script_critic → human_review → (parallel: image_worker + voiceover_worker) → seo → editor_prep → assemble → upload`.

Things to know before editing the graph:
- **Error routing keys on `last_failed_node`, NOT `errors`.** `errors` and `image_paths` are `Annotated[..., operator.add]` accumulators — they grow across checkpoint resumes of the same `thread_id`, so they can't signal "did this node just fail." Each node sets `last_failed_node` to the node name on failure and `""` on success; the `check_*_error` routers and `error_recovery_node` read that field.
- `error_recovery_node` picks an action (`retry` / `fallback` / `skip` / `abort`) and `route_error_recovery` maps it back into the graph. Retrying a parallel worker routes through the `parallel_spawn` dummy fan-out node, not the worker directly.
- The graph compiles with `interrupt_before=["human_review"]`. Human review and the Ghost Editor are **interrupts** — `graph.invoke()` returns early and the run stays paused in SQLite until resumed. `_run_graph_in_background` in `api/routes/pipeline.py` distinguishes "paused at interrupt" (`graph_state.next` is non-empty) from "done" and only emits a completion event in the latter case.
- Runs are keyed by `thread_id = f"run_{run_id}"`. Resuming, retrying, and edits all mutate the same checkpoint thread.

`PipelineRunner` (`core/pipeline_runner.py`) is the thin bridge the GUI and CLI both use: it builds the initial state, creates a per-run output subfolder (`<safe_title>_<timestamp>`), registers a `queue.Queue` with the broadcaster, and kicks off `_run_graph_in_background`.

## Configuration (`config.py` + `core/config_manager.py`)

There are two config layers — don't confuse them:
- **`config.py`** — module-level constants and path resolution (`APP_VERSION`, video dims, FFmpeg discovery, logger factory). `APP_VERSION` must stay in sync with `package.json`, `installer_v4.iss`, and README branding.
- **`core/config_manager.py`** — the runtime settings store (`from core.config_manager import config`). Singleton over `config.json` with **dot-notation** access: `config.get("tts.backend")`, `config.set("pipeline.upload_enabled", True)`. `DEFAULT_CONFIG` in this file is the source of truth for every setting; new defaults are auto-merged into a user's existing `config.json` on load. A human-editable `.env.local` mirror is generated/synced via `ENV_LOCAL_MAP`, and `_validate_v3_fields()` clamps/migrates legacy values.

### Frozen vs. dev paths (critical)
When packaged (`sys.frozen`), the app runs from Program Files and **cannot write beside the EXE**. `config.py::get_writable_path()` and `get_user_data_dir()` redirect `config.json`, `output/`, `temp/`, the FFmpeg cache, and `ghost_runs.db` to `%LOCALAPPDATA%\GhostCreatorAI\`. Any new file the app writes at runtime must go through `get_writable_path()`, never a bare relative path.

## Pluggable backends (`backends/`, `modules/`)

- **TTS** (`backends/tts/`): `omnivoice_tts` (default; external local server at `tts.omnivoice_url`, clones from `my_voice_reference.wav`), `edge_tts` (zero-config), `elevenlabs`. Selected by `config.get("tts.backend")`.
- **Image** (`backends/image/`): `gemini_imagen`. `backends/base.py` defines the backend contract (`validate_config`, async `generate`).
- **Footage** is separate from images: `config.get("documentary.footage_source")` ∈ `stock` (Pexels + yt-dlp), `meta_ai`, `grok`, `ai_images`. Helpers `uses_video_footage()` / `uses_ai_images()` in `config_manager.py` decide which worker path runs.
- `modules/` holds the heavy lifting used by nodes: `researcher`, `scripter`, `documentary_assembler` (FFmpeg concat/mux/subtitle burn), `uploader` (Playwright → YouTube Studio), `thumbnail_maker`, `voicer`, `video_fetcher`, Indic-language TTS number/text normalization.

## Commands

All Python commands assume the venv is active: `call venv\Scripts\activate.bat` (Windows) — the project is Windows-first and uses PowerShell/`.bat`.

```powershell
# Dev GUI (Electron + React + auto-started Python API)
npm run electron:dev          # or: run.bat

# Run the Python API alone (debug backend / missing-lib errors)
python -m api.server          # serves 127.0.0.1:8766; override with GHOST_API_PORT

# Headless pipeline (no review/editor modals — forced off)
python main.py --topic "OpenAI reasoning models" --upload
python main.py --from-video --video-file output/film.mp4

# Tests (pytest; tests/ mocks all external APIs)
pytest                        # whole suite
pytest tests/test_graph_pipeline.py            # one file
pytest tests/test_graph_pipeline.py -k subtitle -v   # one test

# Frontend-only
npm run dev                   # Vite dev server (no Electron)
npm run build                 # tsc electron config + vite build
```

### Building a release (Windows)
```powershell
build-api.bat        # PyInstaller via GhostCreatorAPI.spec → dist-api/GhostCreatorAPI/ (Python sidecar)
build-electron.bat   # vite build + electron-builder → release/
# Then compile installer_v4.iss in Inno Setup for the final installer EXE.
```
`package.json` `extraResources` bundles `dist-api/GhostCreatorAPI` into the Electron app as `resources/GhostCreatorAPI`, which is what `python-bridge.ts` looks for at runtime.

## Conventions & gotchas

- **Windows-first.** Subprocess calls use `CREATE_NO_WINDOW`; paths assume `%LOCALAPPDATA%`. Use PowerShell syntax.
- **FFmpeg is not assumed on PATH.** Always resolve via `config.get_ffmpeg_executable()` / `get_ffprobe_executable()` (checks user cache → beside-EXE → PATH). The packaged app downloads FFmpeg into the user cache on first launch (`ensure_ffmpeg.ps1` / `core/ffmpeg_bootstrap.py`).
- **The OmniVoice TTS stack is intentionally NOT in `requirements.txt`** — it runs as a separate external server; only the HTTP client lives here.
- Indic languages are first-class: `pipeline.language` ∈ `hi hinglish en mr bn gu ta te or`. Image prompts are always generated in English regardless.
- `ghost_runs.db` (+ `-shm`/`-wal`) is the live SQLite checkpoint DB in the repo root during dev — it holds run state, not source.
- Port conflicts on 8766 are the most common "app stuck initializing" cause; `api/server.py` distinguishes "already healthy" (reuse) from "port taken but dead" (error with kill instructions).

---
> Source: [HunterisLive-1/ghost-creator](https://github.com/HunterisLive-1/ghost-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
