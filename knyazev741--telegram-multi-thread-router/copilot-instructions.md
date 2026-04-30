## telegram-multi-thread-router

> Python asyncio bot using aiogram 3 + provider backends. Each Telegram bot thread = one provider session.

# Telegram Multi-Thread Router

## Architecture

Python asyncio bot using aiogram 3 + provider backends. Each Telegram bot thread = one provider session.

- **Bot**: aiogram 3 Dispatcher with Router-per-concern pattern
- **Sessions**: local Claude/Codex runners per thread, managed by SessionManager
- **Orchestrator**: Auto-created provider session with MCP tools for managing other sessions
- **Permissions**: can_use_tool callback → asyncio.Future → Telegram inline buttons
- **DB**: aiosqlite with WAL mode for session/topic persistence
- **Config**: pydantic-settings loading from .env

## Quick Start

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env  # fill in values
python -m src
```

## Restart (pick up new code)

Two ways to restart:
1. **From any existing bot topic**: `/restart`
2. **Manual**: `kill $(pgrep -f 'python -m src') && python -m src`

On restart, all sessions resume automatically via `resume_all()` — sessions stay as `idle` in DB during graceful shutdown and are re-created with their `session_id` on next startup.

**CRITICAL — DO NOT restart the bot from sessions:**
- **NEVER** run `os.execv`, `kill`, `pkill`, or any command that restarts/kills the bot process from within a provider session running inside this bot.
- You ARE running inside this bot. Restarting it kills YOUR OWN process and causes infinite restart loops.
- If code changes need a restart, tell the user to run `/restart` from any existing bot topic manually.
- This applies to ALL sessions, including the orchestrator.

**IMPORTANT**: Never `kill -9` the bot — use SIGTERM so on_shutdown runs and preserves session state.

## Project Structure

```
src/
  __main__.py          - Entry point (asyncio.Runner + uvloop)
  config.py            - pydantic-settings BaseSettings
  bot/
    dispatcher.py      - Dispatcher factory, startup/shutdown lifecycle
    middlewares.py      - OwnerAuthMiddleware
    routers/
      general.py       - General topic fallback (minimal, rarely fires)
      session.py       - All commands (/new, /list, /restart, /stop, /close) + message forwarding
    status.py          - StatusUpdater (editable status message per turn)
    output.py          - split_message, TypingIndicator
  sessions/
    runner.py          - SessionRunner (ClaudeSDKClient wrapper, state machine)
    manager.py         - SessionManager (thread_id → runner mapping)
    permissions.py     - PermissionManager (asyncio.Future bridge to Telegram buttons)
    orchestrator.py    - Auto-created orchestrator session with management MCP tools
    mcp_tools.py       - Telegram output MCP tools (reply, send_file, react, edit_message)
    voice.py           - faster-whisper transcription
    health.py          - Zombie session detection
    state.py           - SessionState enum
    remote.py          - RemoteSession proxy for TCP workers
  db/
    schema.py          - SQLite schema + migrations
    connection.py      - aiosqlite connection helper
    queries.py         - Named SQL query functions
  ipc/
    protocol.py        - msgspec Struct message types for TCP
    server.py          - Bot-side TCP server for remote workers
  worker/
    __main__.py        - Worker entry point (python -m src.worker)
    client.py          - TCP client with reconnection
    output_channel.py  - Bot adapter for worker side
```

## Telegram Bot Commands

**Orchestrator thread (🎯 main interface):**
- Natural language → create/list/stop sessions, toggle auto-mode
- Also a full provider session (SSH, filesystem, commands)
- MCP tools: `create_session`, `list_sessions`, `stop_session`, `auto_mode`

**All commands work from any thread:**
- `/new <name> <workdir> [server] [provider]` — create session in new thread
- `/list` — list active sessions
- `/restart` — restart bot, resume all sessions
- `/stop` — interrupt current turn (like Escape in CLI), session stays alive
- `/close` — kill session + delete thread
- Any other `/command` → forwarded to the active provider (`/model`, `/clear`, `/compact`, etc.)
- Text → forwarded to the active provider
- Voice → transcribed → forwarded
- Photo/Document → downloaded to workdir → path sent to the active provider

**General topic**: ignored (Telegram auto-creates new topics there)

## Security
- All secrets in `.env` (gitignored), chmod 600
- No credentials in source code
- OWNER_USER_ID enforced via outer middleware on all messages
- AUTH_TOKEN for TCP worker authentication

## Multi-server Architecture

The bot supports running provider sessions on multiple machines via TCP IPC workers:
- **Bot (hub)**: handles Telegram, dispatches to workers
- **Workers**: run on remote machines, connect to bot via `python -m src.ipc.client`
- **IPC protocol**: TCP + msgspec, token auth

Worker launch: `python -m src.ipc.client --host <bot-ip> --port 9800 --token $AUTH_TOKEN --worker-id <name>`

---
> Source: [knyazev741/telegram-multi-thread-router](https://github.com/knyazev741/telegram-multi-thread-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
