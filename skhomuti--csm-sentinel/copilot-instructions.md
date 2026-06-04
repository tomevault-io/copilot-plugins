## csm-sentinel

> These are the core guidelines for working in this repository.

# Repository Guidelines

These are the core guidelines for working in this repository.

## Project Structure & Modules
- `sentinel/main.py`: CLI entrypoint; asserts event mappings and hands off to the runtime bootstrapper.
- `sentinel/app/`: Runtime wiring (bootstrap, runtime container, context, persistence helpers).
- `sentinel/handlers/`: Telegram handlers grouped by domain (`admin/`, `start.py`, `tracking.py`, etc.).
- `sentinel/services/`: Long-running services such as the Web3 subscription layer.
- `sentinel/jobs.py`, `rpc.py`, `models.py`: Scheduled jobs, RPC subscriptions, and ABI bindings.
- `sentinel/modules/<module>/events.py`, `sentinel/modules/<module>/texts.py`: Module-specific event definitions and message templates.
- `tests/`: Pytest suite (unit/async tests, mocks).
- `abi/`: Contract ABIs loaded by the app.
- `.storage/`: Local persistence for Telegram state (mounted as a volume in Docker).
- `Dockerfile`, `docker-compose*.yml`: Containerization and local orchestration.
- `.env.sample.*`: Example environment files. Copy to `.env` for local runs.

## Build, Test, and Dev Commands
- Test: `uv run pytest -q` (or `./.venv/bin/pytest -q` if `uv` cannot access its cache)
- Linters: install with `uv add --dev ruff ty` (updates `uv.lock`)
- Lint: `uv run ruff check .`
- Format: `uv run ruff format .`
- Typecheck: `uv run ty check`
- Run locally: `uv run python sentinel/main.py` (requires `.env`).
- Docker: `docker compose up -d` (or `docker compose -f docker-compose-ethd.yml up -d` when co-running with eth-docker).

## Coding Style & Naming
- Python ≥ 3.11; follow PEP 8; 4-space indentation.
- Names: modules/functions `snake_case`, classes `CapWords`, constants `UPPER_SNAKE_CASE`.
- Prefer type hints and small, focused functions. Keep side effects in orchestrators (`main.py`, `app/bootstrap.py`, `rpc.py`, `jobs.py`).

## Testing Guidelines
- Frameworks: `pytest`, `pytest-asyncio`.
- Tests live in `tests/` and are named `test_*.py` with clear, behavior-driven names.
- Mock external I/O (Web3, `aiohttp`, env) using `unittest.mock` and `@patch.dict`.
- Run fast: avoid real network calls; rely on the prebuilt `.venv` via `uv run`.

## Commit & Pull Requests
- Commits: imperative mood, concise title, context in body (what/why), reference issues (e.g., `Closes #123`).
- PRs: include a summary, screenshots/logs of bot output if UI/UX changes, test plan (`uv run pytest -q`), and any env vars introduced/changed.
- Keep diffs minimal and focused; add/update `.env.sample.*` when touching configuration.

## Security & Configuration
- Never commit secrets. Use `.env` locally; base it on `.env.sample.*`.
- Key envs: `TOKEN`, `WEB3_SOCKET_PROVIDER`, contract addresses (CSM/ACCOUNTING/FEE_DISTRIBUTOR/VEBO), and URLs (`ETHERSCAN_URL`, `BEACONCHAIN_URL`, `CSM_UI_URL`).
- Persistence path is `.storage/persistence.pkl` (mounted volume in Docker).
- Admins: `ADMIN_IDS` (comma- or space-separated Telegram user IDs) to restrict admin-only commands.

# Agents Guide

This document captures practical conventions for working with this repo using agent tooling.

## Config & Env Access
- Prefer `get_config()` from `sentinel.config` over ad-hoc `os.getenv(...)`.
- Import once and reuse a module-level `CFG = get_config()`.
- For tests that tweak env vars, call `get_config.cache_clear()` before re-reading.
- When new envs are introduced, add them to `Config`, update `.env.sample.*`, and use `CFG` everywhere.

## Runtime & Handlers
- Use `app.bootstrap.create_runtime()` to wire the bot: attach new dependencies to `BotRuntime` instead of global singletons.
- Access runtime utilities from handlers through `BotContext` (`context.runtime`, `context.bot_storage`, etc.).
- Register new handlers in `handlers/__init__.py` so they are picked up during `register_handlers(runtime)`.
- Prefer pure handler helpers; persist side-effects with the storage helpers in `app.storage`.

## Chain Reads & Versioning
- Always read contract state at the event’s block: pass `block_identifier=event.block`.
- Where pre-state is needed, use `event.block - 1` intentionally and document it in code.

## Logging
- No `print`; use module-level `logger` with structured context.
- Warn rather than fail for recoverable issues (e.g., version probe failures, optional filters).

## Typing
- Use built-in types for unions and generics: `str | None`, `set[int]` (Python ≥ 3.11).
- Prefer precise types on public helpers; keep handlers small and focused.
- Do not make constructor/runtime dependencies optional when they are guaranteed by construction. Pass required dependencies explicitly and let missing wiring fail early; reserve `| None` for real absence states and documented fallback behavior.
- Skip `from __future__ import annotations`; the runtime is already Python 3.11, so use stringified forward refs when needed.

## Adding Events/ABIs
- Place new ABIs under `abi/` and import them in `models.py`.
- Extend the module-specific event registry via decorators in `sentinel/modules/<module>/events.py` and add matching message/description entries in `sentinel/modules/<module>/texts.py`.
- Ensure `rpc.py` subscriptions/topics include the relevant ABIs and addresses from `CFG`.
- Maintain the invariant checked in `main.py` that events, messages, and descriptions are in sync.

---
> Source: [skhomuti/csm-sentinel](https://github.com/skhomuti/csm-sentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
