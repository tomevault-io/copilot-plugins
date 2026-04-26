## sse-playground

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make dev      # start dev server with hot-reload (uvicorn --reload)
make install  # uv sync --all-groups
make fix      # ruff check --fix + ruff format (use this before committing)
make check    # lint + format --check, no writes (CI)
make clean    # remove __pycache__ and .ruff_cache
```

## Architecture

Single-file FastAPI app (`main.py`) that serves three SSE endpoints and a static frontend.

**SSE endpoints** — each returns an `EventSourceResponse` wrapping an `async def generator()` that yields dicts with SSE fields (`data`, `event`, `id`, `retry`, `comment`). The generator checks `await request.is_disconnected()` at the top of every iteration to clean up when the browser closes the tab.

**Log stream queue** — `log_queue: asyncio.Queue` is a module-level singleton. `GET /stream/logs` reads from it; `POST /trigger-log` writes to it. This is the pattern to follow if adding more on-demand push endpoints.

**Frontend** — a single `static/index.html` served via `StaticFiles`. No build step. Uses the browser's native `EventSource` API directly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fellipegpbotelho) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
