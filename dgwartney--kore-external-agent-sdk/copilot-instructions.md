## kore-external-agent-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All common tasks run through `run.sh` or directly via `uv run`:

```bash
./run.sh install       # Install all dependencies (runtime + dev)
./run.sh dev           # Dev server with hot-reload on :8000
./run.sh start         # Prod server (4 workers) on :8000
./run.sh test          # Run full pytest suite
./run.sh lint          # Lint with ruff
./run.sh format        # Auto-format with ruff
```

Run a single test:
```bash
uv run --group dev pytest tests/test_app.py::test_post_returns_200
```

## Architecture

This is a single-file FastAPI service (`app.py`) using a **class-based router pattern**. All routes and handler logic live in the `AgentRouter` class, which registers routes at `__init__` time via `_register_routes()`. There is one endpoint: `POST /`.

**Request/Response flow:**
- Pydantic v2 models enforce strict enum validation at the boundary — invalid payloads are rejected as 422 before reaching handler code
- `AgentRequest` → `AgentRouter.handle_agent_request()` → `AgentResponse`
- `sessionIdentity` array must contain both a `sessionReference` and `userReference` entry; missing either returns 422
- Debug output (thought entries) is only included in the response when `debugMode` is `"full"` or `"thoughts"`

**Key enums:** `DebugMode` (`full`/`no`/`thoughts`), `StreamMode` (`token`/`messages`), `InputType` (`text`), `SessionIdentityType` (`sessionReference`/`userReference`)

**ID generation:** UUIDs truncated to hex prefixes — `messageId` uses 6 chars (`msg-{6}`), session/user/run IDs use 8 chars.

**Environment:** Configured via `.env` (see `.env.example`). Key vars: `HOST`, `PORT`, `WORKERS`, `LOG_LEVEL`, `APP_ID`.

## Tooling

- **Python 3.12**, managed by `uv` (no manual venv activation needed with `uv run`)
- **pytest** with `asyncio_mode = auto` (tests are async by default)
- `uv add <pkg>` / `uv add --group dev <pkg>` to manage dependencies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dgwartney) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
