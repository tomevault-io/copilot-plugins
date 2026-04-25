## discord-bot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the bot
python -m src.bot

# Run all tests
pytest

# Run a single test file
pytest tests/test_sessions.py

# Run a single test by name
pytest tests/test_gateway.py::test_send_returns_response
```

## Architecture

This is a Discord bot that bridges Discord messages to a local **OpenClaw** AI agent via WebSocket.

**Request flow:**
1. `bot.py` receives a Discord message (via `@mention` or `!ask` prefix)
2. Retrieves the user's conversation history from `SessionStore`
3. Sends the message + history to `OpenClawGateway` over WebSocket
4. Streams back streamed result chunks until `"done": true`
5. Appends the exchange to `SessionStore` and replies to Discord

**Key components:**
- `src/gateway.py` — `OpenClawGateway`: WebSocket client for OpenClaw. Serializes messages as `{"method": "sessions_send", "params": {"message": ..., "history": [...]}}`. Streams `{"result": "..."}` chunks until `{"done": true}`. Has exponential-backoff `connect_with_retry()`.
- `src/sessions.py` — `SessionStore`: in-memory per-user conversation history using a bounded `deque`. History is a list of `{"role": "user"|"assistant", "content": "..."}` dicts.
- `src/bot.py` — Discord client. Uses per-user `asyncio.Lock` to prevent concurrent requests from the same user. Splits replies >2000 chars into chunks.
- `src/config.py` — All config loaded from `.env` via `python-dotenv`.

## Configuration

Copy `.env.example` to `.env` and set `DISCORD_TOKEN`. All other values have defaults:

| Variable | Default |
|---|---|
| `OPENCLAW_WS_URL` | `ws://127.0.0.1:18789` |
| `COMMAND_PREFIX` | `!ask` |
| `MAX_HISTORY` | `20` |
| `REQUEST_TIMEOUT` | `30` |

## Testing

Tests use `pytest-asyncio` with `asyncio_mode = auto`. Gateway/integration tests spin up real in-process WebSocket servers (no mocking) on localhost ports 19997–19999. Each test file uses a distinct port to avoid conflicts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/contraconceptions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
