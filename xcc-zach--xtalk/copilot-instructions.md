## xtalk

> DO NOT commit by yourself. Commit changes only on my request.

# Commit Guideline

DO NOT commit by yourself. Commit changes only on my request.

# Repository Guidelines

## Project Structure & Module Organization
- `src/xtalk/` — core Python package
  - `serving/` event bus, websocket input/output gateways, ASR/TTS managers
  - `pipelines/` pipeline interfaces and `DefaultPipeline`
  - `speech/` ASR (`paraformer_local`, `dummy`), TTS (`edge_tts`, `cosyvoice_local`, `index_tts`, `dummy`), captioner (`qwen3_omni_captioner`)
  - `llm_agent/` agents (e.g., `DefaultAgent`) and `llm_model/` (e.g., `LocalQwenChatModel`, `DummyChatModel`)
  - `logging.py` shared logger setup (logs written to `logs/`)
- `frontend/src/` — browser client (VAD, audio capture/playback): `index.js`, `vad-processor.js`, `fastenhancer_s.onnx`
- `examples/sample_app/` — FastAPI demo servers and HTML templates
- `docs/plan.md` — high-level roadmap/notes

## Build, Test, and Development Commands
- Install (editable): `pip install -e .`
- Demo extras (CPU-friendly): `pip install -e .[paraformer-local,edge-tts,server]`
- Run demo server: `python examples/sample_app/dummy_server.py`
- Interactive server: `python examples/sample_app/configurable_server.py`
- Qwen3‑Omni captioning: ensure service at `http://localhost:8901/v1`; `Qwen3OmniCaptioner` accepts http(s) URL, local file path, or bytes (embedded as data URL).
- Lint/format/type-check (via pre-commit hooks):
  - `pre-commit install`
  - `black .` • `ruff check .` • `mypy src`

## Coding Style & Naming Conventions
- Python 3.10+, 4-space indentation, UTF-8.
- Formatting: Black; Linting: Ruff; Typing: mypy-friendly type hints.
- Comments: write code comments in English. Keep inline comments/docstrings concise and specific to the code.
- Logging: write log messages (logger.debug/info/warning/error) in English for clarity and easier debugging.
- Modules and files: `snake_case`; classes: `PascalCase`; constants: `UPPER_SNAKE_CASE`.
- Prefer explicit names over single-letter identifiers (except trivial loops).

## Testing Guidelines
- Framework: pytest (dev extra provides `pytest`, `pytest-cov`).
- Layout: place tests under `tests/`, name files `test_*.py`.
- Run tests: `pytest -q` • Coverage: `pytest --cov src/xtalk`.
- For integration checks, start `dummy_server.py` and verify WebSocket flow end‑to‑end.

## Commit & Pull Request Guidelines
- Commits: imperative mood, concise subject (≤72 chars), meaningful body when needed. Prefer Conventional Commit style (e.g., `feat(serving): ...`).
- PRs: include purpose, scope, testing steps, and any screenshots of frontend changes. Link issues, list breaking changes, and update docs/examples when behavior changes.
- Keep diffs focused; avoid unrelated refactors. Ensure `black`, `ruff`, and `mypy` pass.

## Architecture Overview & Tips
- Event-driven service: `EventBus` routes events between `InputGateway` (WS in), `ASRManager`, `TTSManager`, and `OutputGateway` (WS out).
- Audio flow: frontend streams continuous 16 kHz PCM and sends VAD start/end as segment markers. Backend ASR pre-buffers recent frames (~20) and injects them as padding on VAD start. TTS output commonly 48 kHz.
- For OpenAI-based LLMs, set `OPENAI_API_KEY` and `OPENAI_MODEL`; optional `OPENAI_BASE_URL`.
- Some TTS/ASR require `ffmpeg` installed (see README for platform commands).

---
> Source: [xcc-zach/xtalk](https://github.com/xcc-zach/xtalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
