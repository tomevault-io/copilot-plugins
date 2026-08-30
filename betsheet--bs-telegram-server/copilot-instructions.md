## bs-telegram-server

> A small FastAPI microservice that wraps a private Telethon-based client (`bstelegramuser`) and exposes a REST gateway for Telegram user sessions. Keeps one in-memory `BSTelegramUserClient` per user, persists Telethon sessions under `sessions/<user_id>.session`.

# Agent notes for bs-telegram-server

## What this is
A small FastAPI microservice that wraps a private Telethon-based client (`bstelegramuser`) and exposes a REST gateway for Telegram user sessions. Keeps one in-memory `BSTelegramUserClient` per user, persists Telethon sessions under `sessions/<user_id>.session`.

## Stack & tooling
- **Python**: `>=3.12` (pyproject.toml)
- **Dependencies**: `fastapi[standard]`, `httpx`, `python-dotenv`, `uvicorn`, plus two private GitHub packages: `bstelegramuser` and `bsutils`.
- **Setup**: no `requirements.txt`; use `pyproject.toml`.
- **Task runner / lint / test**: none configured. There is no pytest, ruff, mypy, or CI config in this repo.

## Setup commands
```bash
# With uv (start.sh does this)
uv venv .venv
uv pip install -e .

# With pip
python -m venv .venv
source .venv/bin/activate
pip install -e .
```

## Required environment
Create a `.env` file in the project root. All of these are required:

```dotenv
API_BASE_URL=https://api.betsheet.internal
API_SERVICE_JWT_TOKEN=...
GET_TELEGRAM_APP_CREDENTIALS_ENDPOINT=/get-telegram-app-credentials
PROCESS_TELEGRAM_MESSAGES_ENDPOINT=/pick/process-telegram-messages
RUNNING_PORT=8008   # optional, defaults to 8008
```

`load_dotenv()` is called at import time in `util/config.py` and `util/util.py`.

## Running the server
```bash
# README recommendation (auto-reload)
fastapi dev app.py

# Production
uvicorn app:app --host 0.0.0.0 --port 8008

# Or via the provided bootstrap script
./start.sh
```

`app.py` also runs `uvicorn.run(app, host="localhost", port=RUNNING_PORT)` when executed directly.

## Project structure
- `app.py` — FastAPI app, imports routers, no base path.
- `routers/` — `auth_router.py`, `channels_router.py`, `listening_router.py`. All are included with their `prefix` already set.
- `managers/telegram_client_manager.py` — global singleton `telegram_client_manager`; creates/retrieves `BSTelegramUserClient` instances keyed by `user_id`.
- `services/channels_service.py` — `get_n_messages_kway_merge()` fetches the `n` most recent messages across multiple channels using a k-way merge over lazy async generators.
- `util/config.py` — loads env, exposes `API_BASE_URL`, `CREDENTIALS_ENDPOINT`, `PROCESS_PICK_MESSAGES_ENDPOINT`, `RUNNING_PORT`, `logger`.
- `util/util.py` — `get_telegram_app_credentials()`, `get_telegram_client_manager()`, `ensure_session_exists()`, `verify_client_authentication_state()`.
- `test.py` — a manual, ad-hoc script that copies a session to a temp file and replays the last N messages from hardcoded channels against the processing endpoint. **Not an automated test suite.**

## Things an agent is likely to miss
- **Telegram API credentials are not in `.env`**. They are fetched at runtime from `API_BASE_URL + GET_TELEGRAM_APP_CREDENTIALS_ENDPOINT` using `API_SERVICE_JWT_TOKEN`.
- **Session directory is runtime state**. `sessions/` is gitignored. The server creates it as needed. Do not commit session files.
- **A session can only be used by one process at a time**. Telethon locks the SQLite session file. `test.py` works around this by copying `sessions/<user_id>.session` to a temp file before opening it.
- **No real tests exist**. If asked to run tests, look for `pytest`/`unittest` — there are none. `test.py` is a manual smoke script with hardcoded credentials and channels.
- **Start script uses uv**. `start.sh` installs `uv` if missing, then `uv pip install -e .` before running `python app.py`. It does not run any checks.
- **README is the API contract**. `README.md` documents the endpoints, request/response shapes, and authentication flow. Trust it over comments for public API behavior.
- **Code comments are in Spanish**, but the public API in README is in English.
- **pyproject.toml excludes `sessions*`, `routers*`, `util*`** from the setuptools package discovery, so the install is effectively editable-mode-only for these local modules.

## Verification order (if changing things)
There is no formal lint/typecheck/test pipeline. After edits, verify by running the server and exercising the endpoints locally:

```bash
fastapi dev app.py
# Then hit the endpoints in the README curl example.
```

For manual verification of message processing, use `test.py` (requires a real `.env` and an existing session file).

---
> Source: [betsheet/bs-telegram-server](https://github.com/betsheet/bs-telegram-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
