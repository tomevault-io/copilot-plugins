## subtext

> This file is the fast context guide for coding agents and contributors working on Subtext.

# AGENTS.md - Subtext Contributor Guide

This file is the fast context guide for coding agents and contributors working on Subtext.

## Project Identity

Subtext is one local-first project with two companion modes:

- **Private web service**: iPhone/browser workflow for private URL download, URL transcription, and local file transcription over Tailscale
- **Desktop app**: full PySide workflow for local media processing, transcript review, Ollama analysis, and exports

Keep these modes coherent, but do not blur their boundaries in code or docs.

## Project Goals

- Keep setup simple for non-technical users.
- Prefer local-first processing and private-network access.
- Preserve UI responsiveness during heavy desktop work.
- Keep the private web service fast and dependable for repeat phone use.
- Favor clear status messages, safe fallbacks, and memory-stable defaults.

## Current Product Boundaries

### Private Web Service

- Runs via `run_web.py`
- Binds to `127.0.0.1:8000` by default
- Intended to be exposed privately through Tailscale Serve
- Supports:
  - URL transcription
  - local audio/video file transcription
  - URL video download
- Does **not** include the Desktop app’s transcript editing, Ollama analysis, or export workflow

### Desktop App

- Runs via `run.py`
- Uses PySide6 tabs and QThread workers
- Supports:
  - queued local + URL workflows
  - transcript review/editing
  - Ollama analysis
  - export formats

Do not describe or implement the web service as if it already has full desktop AI analysis features unless that is an explicit new feature request.

## Stack

- Python 3.11+
- PySide6 for desktop UI
- FastAPI for the private web service
- AsyncIO + QThread workers
- yt-dlp for media download/captions
- Whisper / optional faster-whisper for transcription
- Ollama for local LLM analysis
- `uv` for dependency management

## Code Boundaries

- `src/ui/`: Desktop Qt UI only
- `src/ui/workers/`: QThread wrappers for desktop async calls
- `src/web/`: private FastAPI service + static web UI
- `src/core/`: business logic and integrations; no direct UI updates
- `src/config/`: shared config and path constants
- `scripts/`: install/start/maintenance helpers
- `docs/`: human-facing technical/product docs

Do not place heavy business logic directly in Qt tab classes or browser-facing JS.

## Key Runtime Rules

1. Desktop UI updates must happen in the UI thread.
2. Long desktop tasks must run in workers.
3. Workers should communicate with signals only.
4. Async I/O belongs in core or web service modules, not Qt widgets.
5. Always use `ProjectPaths` from `src/config/paths.py`.
6. Keep the private web service localhost-only unless the user explicitly requests a different security model.
7. Prefer Tailscale/private-network patterns over public exposure.

## Current Processing Behavior

- Input can mix URLs and local files in the desktop app.
- YouTube URLs can use captions-first fast path.
- If captions fail, are unavailable, or rate-limit, fallback is Whisper.
- Batch work is sequential by design for memory stability.
- Desktop processing unloads Whisper resources after runs.
- Private web service keeps the transcription model warm across requests.
- Ollama calls use low-memory behavior and should remain desktop-focused unless intentionally expanded.

## Private Web Service Expectations

- Entry point: `run_web.py`
- Main service: `src/web/server.py`
- Static client: `src/web/static/`
- Auth model:
  - shared secret via `SUBTEXT_SERVER_KEY`
  - optional IP allow rules
  - Tailscale is the intended access path
- Current web UX is mobile-first and optimized for Safari on iPhone.
- Browser download flows should prefer attachment-style responses over fragile JS-only download tricks.

When changing the web service:

- Preserve localhost binding by default.
- Do not reintroduce old LAN/`0.0.0.0`/same-Wi-Fi assumptions in code or docs.
- Keep the Safari/iPhone path smooth.
- Treat download and transcription as first-class web tasks.

## Desktop App Expectations

- Main app shell: `src/ui/main_window.py`
- Media workflow: `src/ui/download_tab.py`
- AI workflow: `src/ui/analysis_tab.py`
- Results/export: `src/ui/results_tab.py`

When changing desktop behavior:

- Keep heavy work out of widgets.
- Preserve signal-driven state transitions.
- Keep transcript-to-analysis flow intact.

## Common Feature Entry Points

### New desktop download/transcript option

- `src/ui/widgets/multi_select_dropdown.py`
- `src/ui/download_tab.py`
- `src/ui/workers/download_worker.py`
- `src/core/processor.py`

### New desktop AI analysis feature

- `src/core/analyzer.py`
- `src/ui/workers/analysis_worker.py`
- `src/ui/analysis_tab.py`
- `src/ui/results_tab.py`

### New private web service feature

- `src/web/server.py`
- `src/web/static/index.html`
- `src/web/static/app.js`
- `src/web/static/style.css`
- `src/core/downloader.py` / `src/core/transcriber.py` as needed

## Conventions

- Type hints required on public methods/functions.
- Keep methods focused and reasonably short.
- Prefer explicit names over abbreviations.
- Keep docstrings concise and practical.
- Use `pathlib.Path`, not raw path strings.
- Avoid broad `except:`; catch specific exceptions where possible.
- Prefer ASCII unless the file already uses non-ASCII.

## Dependency and Tooling

- Use `uv sync` to install.
- Use `uv run ...` to execute project scripts.
- Keep dependency specs in `pyproject.toml`.
- If adding optional performance paths, keep the default path simple and well documented.

## Launchers and Setup Helpers

- `mac-run.command` and `win-run.bat` are user-facing launchers and should match the README wording.
- `scripts/com.subtext.private-web.plist` is a tracked LaunchAgent template, not a place for real secrets.
- `scripts/install_launchd.sh` should stay aligned with the real plist filename, port, logs, and current launchctl flow.
- Never commit a real `SUBTEXT_SERVER_KEY` into tracked files.

## Documentation Expectations

When user-facing behavior changes, update:

1. `README.md`
2. `docs/ARCHITECTURE.md`
3. `docs/SPEC.md` if the product framing or primary workflows changed
4. `AGENTS.md` when contributor-facing workflow or conventions changed

Docs should consistently describe:

- the two companion modes
- the boundary between web service and desktop AI features
- the current Tailscale-first private access model

## Testing Expectations

Before finalizing meaningful changes, validate what applies:

1. Desktop app launches: `uv run python run.py`
2. Private web service launches: `uv run python run_web.py`
3. Local health check works: `curl http://127.0.0.1:8000/health`
4. One URL processing flow
5. One local file flow
6. One web download-only flow if web download behavior changed
7. One desktop analysis run if Ollama-related code changed and Ollama is available
8. One export path if desktop results/export behavior changed

If something cannot be tested locally, clearly note it.

## Safe Defaults

- Prefer memory-safe behavior over maximum throughput.
- Prefer fallback behavior over hard failure.
- Prefer private-network access over public exposure.
- Prefer clear status/log messages over silent retries.
- Prefer incremental UX improvements over introducing parallel product paths with overlapping meaning.

## Drift to Watch For

Common sources of confusion in this repo:

- outdated launcher copy that still implies LAN/same-Wi-Fi web access
- docs that oversell the web service as if it includes desktop AI analysis
- helper scripts drifting away from current LaunchAgent names or current ports
- secrets accidentally being placed in tracked plist/templates

Part of the job is keeping these aligned when behavior changes.

---
> Source: [pattty847/Subtext](https://github.com/pattty847/Subtext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
