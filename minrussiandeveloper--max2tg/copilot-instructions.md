## max2tg

> This repository contains the bridge-specific code for `Max2TG`, a bidirectional MAX to Telegram Topics bridge. The main runtime is [main.py](main.py), with MAX integration isolated in [max_bridge.py](max_bridge.py) and SQLite mapping logic in [database.py](database.py). Configuration loading lives in [config.py](config.py).

# Repository Guidelines

## Project Structure & Module Organization

This repository contains the bridge-specific code for `Max2TG`, a bidirectional MAX to Telegram Topics bridge. The main runtime is [main.py](main.py), with MAX integration isolated in [max_bridge.py](max_bridge.py) and SQLite mapping logic in [database.py](database.py). Configuration loading lives in [config.py](config.py).

Operational scripts and utilities are top-level Python files such as `init_max.py`, `complete_max_auth.py`, `rebuild_topics.py`, and `cleanup_topics.py`. Shell helpers are in `scripts/`. Tests are in `tests/`. Runtime data, credentials, logs, and vendored dependencies belong in ignored paths such as `.env`, `data/`, `logs/`, and `vendor/`.

## Build, Test, and Development Commands

Install bridge dependencies locally:

```bash
pip install aiogram aiosqlite python-dotenv
```

Install the external `MaxAPI` dependencies in its checkout:

```bash
cd ../MaxAPI && pip install lz4 msgpack
```

Run the bridge locally with:

```bash
./scripts/start_bot.sh
```

Stop it with `./scripts/stop_bot.sh`. For containers, use `docker compose build` and `docker compose up -d`. Follow runtime logs with `tail -f logs/bridge.log`.

Run tests with:

```bash
python3 -m unittest discover -s tests
```

## Coding Style & Naming Conventions

Use Python 3 style with 4-space indentation and type hints where they clarify async or database boundaries. Keep async functions explicit and avoid blocking calls in Telegram or MAX polling paths. Name modules and functions in `snake_case`; constants and environment variables use `UPPER_SNAKE_CASE`.

Keep bridge logic in the bridge modules rather than copying protocol code from `MaxAPI`. Prefer structured SQLite access through `BridgeDB` instead of ad hoc SQL from unrelated files.

## Testing Guidelines

Tests use the standard library `unittest` framework with async tests via `unittest.IsolatedAsyncioTestCase`. Name test files `test_*.py` and test methods `test_*`. Add focused database tests for mapping or idempotency changes, and add integration-style checks only when external services are not required.

## Commit & Pull Request Guidelines

Recent commits use concise, imperative summaries such as `Stabilize Docker bridge workers and fix Max media upsert`. Keep the first line specific and under roughly 72 characters when possible. Pull requests should explain the behavior change, list any database/runtime impact, mention required `.env` or Docker changes, and include test results. For visible Telegram/MAX behavior changes, include screenshots or log excerpts when useful.

## Security & Configuration Tips

Never commit `.env`, auth bundles, SQLite runtime databases, logs, or a local `vendor/MaxAPI` checkout. Configure `MAXAPI_REPO_PATH` when using a nonstandard `MaxAPI` location. Preserve the worker-process isolation described in `README_BRIDGE.md` unless the GOST TLS crash risk has been revalidated.

---
> Source: [MinRussianDeveloper/Max2TG](https://github.com/MinRussianDeveloper/Max2TG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
