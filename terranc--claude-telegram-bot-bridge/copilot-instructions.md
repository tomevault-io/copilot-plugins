## claude-telegram-bot-bridge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Telegram bot that integrates with Claude Code SDK to run Claude Code sessions and skills from Telegram. Written in Python 3.11+, fully async.

## Code Style

- All code, comments, variable names, shell messages, and log strings must be in **English**
- Bot preset strings (command responses, status messages, error prompts) should be kept **minimal and in English**
- The LLM (Claude) handles language adaptation naturally based on the user's conversation language — no hardcoded i18n needed
- Openspec's documentation is in **Chinese**
- Communication in Claude Code is in **Chinese** as much as possible

```bash
# Setup
python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt

# Run (foreground, default)
./start.sh --path /absolute/path/to/project

# Run (daemon/background)
./start.sh --path /absolute/path/to/project -d

# Debug mode (verbose logging + chat file logging)
./start.sh --path /absolute/path/to/project --debug

# Direct Python invocation
python -m telegram_bot --path /absolute/path/to/project --debug

# Lifecycle
./start.sh --path /path --status
./start.sh --path /path --stop
./start.sh --path /path --install    # macOS launchd auto-start
./start.sh --path /path --uninstall
```

Tests use Python `unittest` under `tests/`.

## Architecture

```
__main__.py          CLI entry → parses --path/--debug → calls bot.run()
    │
core/bot.py          TelegramBot — command handlers, access control,
    │                permission gating, response formatting, message queuing
    │
core/project_chat.py ProjectChatHandler — per-user long-lived Claude SDK
    │                streams, message queueing, tool permission callbacks,
    │                response processing, session history browsing
    │
core/streaming.py    StreamingMessageHandler — progressive draft message updates
    │                for real-time AI response streaming
    │
utils/audio_processor.py AudioProcessor — format detection, ffmpeg conversion,
    │                   temp/stale audio cleanup
    │
utils/transcription.py WhisperTranscriber — OpenAI Whisper calls, retry/backoff,
    │                  empty-text validation, cost/duration logging
    │
session/             SessionManager/SessionStore — per-user state in JSON
    │                (PROJECT_ROOT/.telegram_bot/sessions.json)
    │
utils/config.py      Pydantic Settings config from .env
utils/chat_logger.py Per-session debug chat logging
```

### Key data flows

- **User message** → `TelegramBot` handler → access check → `ProjectChatHandler.process_message()` → Claude SDK stream → response back to Telegram
- **Voice message** → `TelegramBot._handle_voice_message()` → download → format detect/convert → Whisper transcription → `🎤 Voice:` preview → same `process_message()` flow as text
- **Permission gating**: Tool requests pass through `_permission_callback()` in bot.py. File access inside `PROJECT_ROOT` is auto-allowed; outside requires user confirmation via Telegram inline buttons with three options: Allow (once), Deny, or Allow All (session-wide). User responses are handled via asyncio.Future pattern with 60-second timeout.
- **Per-user streams**: Each user gets a persistent `ClaudeSDKClient` connection in `ProjectChatHandler._streams`. Streams are reused across messages and cleared on `/new` or model change.
- **Session state**: Three layers — Telegram-side (SessionStore JSON), SDK-side (~/.claude/projects/{name}/*.jsonl), runtime tracking (_runtime_active_sessions dict).

### Important patterns

- **Progressive streaming**: AI responses are streamed in real-time using Telegram draft messages. Text updates every 150 characters or 1 second. Long messages (>4000 chars) automatically split into multiple drafts.
- **Priority stop command**: `/stop` has priority handling - it bypasses the message queue limit and immediately cancels the currently executing task via `asyncio.Task.cancel()`. This allows users to interrupt runaway executions even when the queue is full.
- **Priority revert command**: `/revert` also has priority handling - bypasses message queue limit, cancels active operations (streaming, voice transcription), and allows users to restore conversation state even during execution.
- **Streaming interruption**: `/stop` and `/new` commands immediately cancel ongoing streaming and delete draft messages. Other commands (`/model`, `/resume`, `/skills`) do not interrupt streaming.
- **Voice interruption**: `/stop` and `/new` cancel active voice transcription tasks and clean temporary audio files.
- **Task cancellation flow**: When `/stop` is invoked, it cancels the active task stored in `_active_tasks[user_id]`. The cancelled task raises `asyncio.CancelledError` in `ProjectChatHandler.process_message()`, which triggers cleanup (delete drafts, stop SDK stream) before returning.
- **Revert flow**: `/revert` displays paginated message history (last 50 messages) via inline keyboard. User selects a message, then chooses from 5 modes: (1) Restore code and conversation, (2) Restore conversation only, (3) Restore code only, (4) Summarize from here, (5) Cancel. Revert truncates SDK JSONL file to selected message, clears active stream, cancels pending tasks, and resets session state. Callback data format: `revert:select:{index}`, `revert:page:{page}`, `revert:mode:{index}:{mode}`.
- **Permission request UI**: When tools need to access paths outside PROJECT_ROOT, a standalone message with inline keyboard buttons is sent. Users can Allow (once), Deny, or Allow All (session-wide). Requests timeout after 60 seconds. The `/new` command clears the approve-all flag.
- **Permission callback pattern**: Uses asyncio.Future to wait for user response. Callback data format: `perm:allow:{request_id}`, `perm:deny:{request_id}`, `perm:allow_all:{request_id}`. Futures are stored in `_pending_permission_futures` dict and resolved by callback handlers.
- `AskUserQuestion` tool is degraded: converted to plain text with numbered options rendered as Telegram inline keyboard buttons.
- Responses with file paths matching media extensions are auto-sent as Telegram photos/documents.
- Message queue per user: max 3 concurrent tasks with overflow rejection. Priority commands like `/stop` and `/revert` bypass this limit.
- `start.sh` handles venv creation, dependency caching (MD5-based), log rotation (14 days), auto-restart with crash detection (>5 in 60s).

## Key environment variables

| Variable | Required | Description |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Yes | Telegram Bot API token |
| `ALLOWED_USER_IDS` | No | Comma-separated user IDs; empty = allow all |
| `CLAUDE_CLI_PATH` | No | Absolute path to Claude CLI binary |
| `CLAUDE_PROCESS_TIMEOUT` | No | SDK timeout in seconds (default: 600) |
| `PROXY_URL` | No | HTTP proxy; start.sh auto-configures env vars |
| `PROJECT_ROOT` | Set by start.sh | Base path for all file access validation |
| `DRAFT_UPDATE_MIN_CHARS` | No | Min characters before draft update (default: 150) |
| `DRAFT_UPDATE_INTERVAL` | No | Min seconds between draft updates (default: 1.0) |
| `OPENAI_API_KEY` | Voice only | API key for Whisper transcription |
| `OPENAI_BASE_URL` | No | Optional OpenAI-compatible Whisper API base URL |
| `WHISPER_MODEL` | No | Whisper model name (default: `whisper-1`) |
| `MAX_VOICE_DURATION` | No | Max accepted voice duration in seconds (default: 300) |
| `FFMPEG_PATH` | No | Optional absolute ffmpeg path |

## Runtime directories

All bot data writes to `PROJECT_ROOT/.telegram_bot/`:
- `sessions.json` — user session persistence
- `logs/bot.log` — main log (daily rotation)
- `logs/error_YYYYMMDD.log` — error log
- `logs/{user_id}_{session_id}_{date}.log` — debug chat logs

SDK conversation logs: `~/.claude/projects/{PROJECT_DIR_NAME}/*.jsonl`

## Version Release Preference
<!-- github-push-and-release: release -->
This project uses full release mode: CHANGELOG + commit + git tag + GitHub release.
To force a full release, use `/github-push-and-release release`.

## Git Commit Language Preference
<!-- github-push-and-release-commit-lang: en -->
Git commit messages use English.

---
> Source: [terranc/claude-telegram-bot-bridge](https://github.com/terranc/claude-telegram-bot-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
