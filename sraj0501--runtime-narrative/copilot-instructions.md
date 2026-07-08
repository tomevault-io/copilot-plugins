## runtime-narrative

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development practices

These rules are non-negotiable for all work in this repository.

**No hardcoded environment values.** Every configurable value — URLs, model names, timeouts, feature flags — must be read from environment variables or a `FailureDiagnosticsConfig`-style config object. Never embed literals that differ between environments directly in code.

**All functions async by default.** Write `async def` unless the function genuinely cannot be async (e.g. a `__exit__` protocol method, a `ContextVar` accessor, or a pure data transformation with no I/O). When wrapping a sync-only third-party call, use `asyncio.to_thread`. Document the reason in a comment when sync is unavoidable.

**Distribute work across subagents and run in parallel.** When implementing a task that has independent parts — separate files, separate test suites, separate concerns — spawn subagents and run them concurrently rather than sequentially. Never do serially what can be done in parallel.

**Git: never push directly to `main`.** Always work on `dev` (create it if it doesn't exist). Push to `dev`, then open a PR to `main`. The PR requires human approval before merging. This applies to all commits — features, fixes, docs, and chores alike.

**Do not reference this file (or any internal guidance file) in code, docs, or wikis.** CLAUDE.md is an internal tool for AI assistance only. It must not appear in source code comments, README, wiki pages, or any user-facing content.

---

## Commands

```bash
# Install dependencies (includes typer, uvicorn, fastapi, pydantic for examples)
uv sync --group dev

# Copy env var template (all vars are optional)
cp .env.example .env

# Run examples
uv run python examples/basic.py
uv run python examples/success.py
uv run python examples/basic_ollama.py  # requires RUNTIME_NARRATIVE_MODEL

# Auto-instrumentation examples (Phase 1)
uv run python examples/narrative_class.py
uv run python examples/instrument_module.py
uv run python examples/auto_instrument.py

# Run FastAPI demo
uv run python -m examples.fastapi_app.run
# With Ollama failure analysis:
RUNTIME_NARRATIVE_MODEL=llama3 uv run python -m examples.fastapi_app.run
# With a custom endpoint (vLLM, llama.cpp, etc.):
RUNTIME_NARRATIVE_MODEL=llama3 RUNTIME_NARRATIVE_ENDPOINT=http://localhost:8000/api/generate uv run python -m examples.fastapi_app.run

# Rich failure diagnostics (locals, etc.) via env
RUNTIME_NARRATIVE_FAILURE_DIAGNOSTICS=rich uv run python examples/basic.py

# Run all tests
uv run pytest tests/ -v

# Run a single test file
uv run pytest tests/test_story.py -v

# Run a single test by name
uv run pytest tests/test_diagnostics.py -v -k "test_name"
```

## Environment variables

| Variable | Values | Default | Effect |
|---|---|---|---|
| `RUNTIME_NARRATIVE_ENV` | `development`, `production` | `development` | Production caps traceback to 8 000 chars and forces lean mode |
| `RUNTIME_NARRATIVE_FAILURE_DIAGNOSTICS` | `lean`, `rich` | `lean` | `rich` captures locals for up to 2 frames |
| `RUNTIME_NARRATIVE_ALLOW_RICH_IN_PRODUCTION` | `1`, `true` | off | Bypass production safeguard for rich diagnostics |
| `RUNTIME_NARRATIVE_MODEL` | model name string | — | Used by example scripts to pick an Ollama/LLM model |
| `RUNTIME_NARRATIVE_ENDPOINT` | URL | — | Custom LLM endpoint for example scripts |
| `RUNTIME_NARRATIVE_RICH_LOG_FILE` | file path | — | Read by `renderer_defaults.default_renderers()` (shared by all auto-instrumentation entry points). Adds a `ConsoleRenderer` writing to this file, on top of the TTY-based base selection |
| `RUNTIME_NARRATIVE_RICH_LOG_CONSOLE` | `1`, `0` | `1` | With `RUNTIME_NARRATIVE_RICH_LOG_FILE` set and stdout a TTY, `0` suppresses the console copy so the narrative goes to the file only |

## Architecture

The library (`runtime_narrative/`) models execution as **stories** composed of **stages**, emitting lifecycle events that renderers consume.

### Core execution flow

1. `story(name)` — dual sync/async context manager (`with` / `async with`) that creates a `StoryRuntime`, sets it on `current_story` (a `ContextVar`), and emits `StoryStarted`/`StoryCompleted` events. On exception, builds enriched failure data (see **Diagnostics** below), optionally runs failure analysis, and emits `FailureOccurred` before `StoryCompleted`. Accepts `module: str | None = None`; when omitted, the calling module is auto-detected via a single `sys._getframe(1)` lookup at construction time (cheap — no stack walking) and stored on `StoryStarted.module`. Wrappers that open `story()` on a caller's behalf (decorators, middleware) should pass `module=` explicitly — `@runtime_narrative_story` passes `func.__module__` for this reason.
2. `stage(name)` — dual sync/async context manager that must be nested inside a `story`. Registers a `StageRecord` on the `StoryRuntime`, manages a `current_stage_stack` ContextVar for nesting, and emits `StageStarted`/`StageCompleted` via **`emit()`** (sync). Nested `stage()` inside `async with story` does not use `emit_async`; async renderers' `handle` coroutines are therefore **not** awaited for stage events (use a sync renderer for full stage coverage, or rely on async emit for story/failure-only paths). Same `module` auto-detection/override as `story()`, stored on `StageRecord.module` and forwarded to `StageStarted.module`.
3. **Context** (`context.py`) — two `ContextVar`s: `current_story` (holds the active `StoryRuntime`) and `current_stage_stack` (holds a list of nested `StageRecord`s). This enables propagation across sync/async without parameter threading.
4. **Events** (`events.py`) — plain dataclasses: `StoryStarted`, `StageStarted`, `StageCompleted`, `FailureOccurred`, `StoryCompleted`, `LLMAnalysisReady`. `FailureOccurred` includes diagnostics fields (`diagnostics_mode`, `primary_frame_reason`, `stack_frames`, `source_snippet`, `compressed_stack_summary`, `hidden_frame_count`, `traceback_truncated`, optional `locals_by_frame`, `redaction_removed_keys`). `StoryStarted`/`StageStarted` include `module: str = ""`. `StoryRuntime.emit()` dispatches synchronously; `StoryRuntime.emit_async()` dispatches asynchronously, awaiting renderers whose `handle` is a coroutine function.
5. **Failure** (`failure.py`) — `summarize_exception()` still produces a minimal `FailureSummary` from a traceback leaf frame. The **story** path uses **`build_enriched_failure()`** in `diagnostics.py`, which picks a **primary** frame (innermost **app** code by default), optional file snippet, structured stack, traceback caps in production, and **rich** locals when enabled.
6. **Diagnostics** (`diagnostics.py`) — `FailureDiagnosticsConfig` (with `from_env()` / `merge()`), `effective_diagnostics_mode()` (forces **lean** in production unless `allow_rich_in_production`), `build_enriched_failure()` → `EnrichedFailure` → `_make_failure_occurred()` in `story.py`. Story kwargs: `diagnostics_config`, `runtime_environment`, `failure_diagnostics`, `allow_rich_in_production`, `app_roots`.
7. **Renderers** (`runtime_narrative/renderer/console.py`) — `ConsoleRenderer` handles all six event types. Uses `typer.secho` for color if `typer` is available, falls back to `print`. Renders LLM analysis in a terminal box via `_render_box`. Prints diagnostics snippet, stack summary, and rich locals when present. Every line is prefixed with a `YYYY-MM-DD HH:MM:SS.mmm` timestamp (built into `_story_tag()`, the single shared tag-building method all seven event branches call). Shows `(module)` on `Story started` lines always, and on `Stage started` lines only when it differs from the previous stage's module for that story (`self._last_module: dict[story_id, str]`, cleared on `StoryCompleted`) — avoids repeating the tag for a run of same-module stages while still surfacing cross-module transitions. Accepts `output=` (any writable file-like object, default `sys.stdout`) to write the same rich, human-readable format to a file instead of the terminal; `typer`/`click` auto-strip ANSI color codes for non-tty streams. **`json_renderer.py`** emits extended `FailureOccurred` fields including `traceback_text` when present, plus `module` on `StoryStarted`/`StageStarted`. **`coalescing_renderer.py`** — `CoalescingRenderer` wraps another renderer and collapses a run of identical back-to-back `StageStarted`/`StageCompleted` pairs (e.g. a status-polling loop) into one `LogRecorded` summary line after the first `threshold` occurrences, to avoid flooding human-facing output; only wrap renderers meant for humans, not machine-consumption renderers that need full fidelity.
8. **Analyzers** (`runtime_narrative/analyzers/ollama.py`) — `LLMFailureAnalyzer` (OpenAI-compatible `/v1/chat/completions`) and `OllamaFailureAnalyzer` (Ollama native `/api/generate`). Both expose `analyze_failure()` (sync, uses `urllib`) and `analyze_failure_async()` (async, runs the sync version via `asyncio.to_thread`). They receive `FailureSummary` from **`EnrichedFailure.summary`** (primary frame + possibly truncated traceback). `story.__aexit__` prefers `analyze_failure_async` when available; falls back to `asyncio.to_thread(analyze_failure)` for third-party analyzers that only provide a sync method.
9. **Background analysis** — `story(..., background_analysis=True)` emits `FailureOccurred` (with `llm_analysis=None`) immediately, then schedules `_run_background_analysis()` as an `asyncio.Task`. When the task completes it emits `LLMAnalysisReady`. The initial `FailureOccurred` includes the same diagnostics fields as the synchronous-analysis path.
10. **Async story exit** — `story.__aexit__` (async) runs `build_enriched_failure` via **`asyncio.to_thread`** so traceback walking and optional file reads do not block the event loop.
11. **Middleware** (`middleware.py`) — `RuntimeNarrativeMiddleware` wraps each request in **`async with story(...)`** (not sync `with`), forwarding `renderers`, `failure_analyzer`, and diagnostic kwargs. Ensures async renderers are awaited for request-scoped stories. When `renderers` is omitted, it (and the Django middleware, Celery `NarrativeTask`, and gRPC interceptors) fall back to the shared **`renderer_defaults.default_renderers()`** — TTY → `ConsoleRenderer`, non-TTY → `JsonRenderer`, plus an optional `RUNTIME_NARRATIVE_RICH_LOG_FILE`-backed `ConsoleRenderer`.
12. **Decorators** (`decorators.py`) — `@runtime_narrative_story` and `@runtime_narrative_stage` wrap sync functions with `with` and async functions with `async with`. `@runtime_narrative_story` forwards diagnostic and `background_analysis` kwargs into `story()`.

### Key design constraints

- `stage()` raises `RuntimeError` if called outside an active `story()`. The story context is never implicit.
- `RuntimeNarrativeMiddleware` is conditionally imported at package level — it requires `starlette` and will silently be absent if that optional dependency is not installed.
- `typer` is an optional dependency (`console` extra). `ConsoleRenderer` falls back to `print` when absent.

### Adding a custom renderer

Implement a class with `handle(self, event: object) -> None` (sync) or `async def handle(self, event: object) -> None` (async) and pass it to `story(..., renderers=[MyRenderer()])`. No base class or registration required. Async renderers are only awaited when using `async with story(...)` via `emit_async()` (including per-request stories created by `RuntimeNarrativeMiddleware`).

### Adding a custom failure analyzer

Implement a class with `analyze_failure(*, story_name, stage_name, failure, stage_timeline, progress_percent) -> str | None`. Optionally add `analyze_failure_async(...)` with the same signature returning `Awaitable[str | None]`; if absent, the sync method is called via `asyncio.to_thread`. Pass the instance as `failure_analyzer=` to `story()`, middleware, or the `@runtime_narrative_story` decorator.

### Test helpers

`tests/conftest.py` provides `CapturingRenderer` (sync) and `AsyncCapturingRenderer` (async) — both collect emitted events in `.events` for assertions without any real output.

---
> Source: [sraj0501/runtime_narrative](https://github.com/sraj0501/runtime_narrative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
