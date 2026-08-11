## python-selma

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
uv sync

# Initialize workspace and copy default skills
uv run setup.py

# Install browser tool support
uv run playwright install chromium

# Run the gateway (REST API on :8000)
uv run gateway.py

# Run all pytest-based unit tests
uv run pytest

# Run integration test scripts individually
uv run test_runtime.py
uv run test_agent_session_chat.py   # interactive CLI chat
uv run test_webchat.py
uv run test_webchat_http.py
uv run test_skills.py
uv run test_function_call.py
```

## Architecture

Selma is an agentic AI gateway that routes user messages from multiple channels (WebChat, Telegram) through a three-layer agent engine and delivers streamed responses back.

### Three-Layer Agent Engine (`runtime.py`)

```
Layer 1: agent_command()          — orchestration: who, which model, which session
Layer 2: run_embedded_pi_agent()  — robustness: retry loop, context overflow, compaction
Layer 3: run_embedded_attempt()   — execution: build system prompt, create session, invoke tools
```

Error classification (`detect_attempt_error`) drives automatic recovery:
- `context_overflow` → compact session → retry (max 3 attempts)
- `thinking_not_supported` → lower thinking level → retry
- `aborted` → surface to caller

### Message Flow

```
User (Telegram / WebChat)
  → gateway.py (FastAPI)
      → CommandManager (/commands)
      → process_message_flow()
          → normalize() → NormalizedTurnInput
          → runtime.agent_command()  (three layers)
          → DeliveryContext callbacks → channel output
```

### Session Persistence

Sessions are keyed by `session_key` and stored in `~/.selma/agents/main/sessions/` (or `.selma/` in cwd, or `$SELMA_STATE_DIR`):
- `sessions.json` — map of session_key → SessionRecord (model, thinking level, skills snapshot, last interaction)
- `<session_id>.jsonl` — full transcript, managed by `my_mono.agent_session`

Sessions reset daily at a configured hour or after idle timeout.

### System Prompt Assembly (`system_prompt.py`)

Built per turn from workspace context files loaded in order:
`AGENTS.md → SOUL.md → IDENTITY.md → USER.md → TOOLS.md → MEMORY.md → HEARTBEAT.md → daily memory → BOOTSTRAP.md`

Plus: skills snapshots (SKILL.md files, version-hashed and cached in SessionRecord), active tool names, runtime info (model, channel, host).

### Channel Adapters

`channel_adapter.py` defines the `ChannelAdapter` protocol. Implementations:
- `adapter_webchat.py` — Server-Sent Events streaming via asyncio queue
- `adapter_telegram.py` — Telegram bot via python-telegram-bot

### Key Components

| File | Role |
|------|------|
| `gateway.py` | FastAPI app, SSE endpoints, heartbeat lifespan |
| `runtime.py` | Three-layer engine, session management, error recovery |
| `config.py` | Pydantic models loading `.selma/selma.json` |
| `session_store.py` | SessionRecord persistence |
| `system_prompt.py` | System prompt builder |
| `tools.py` | Web search (DuckDuckGo), web fetch (trafilatura), browser (Playwright), file tools |
| `skills.py` | SKILL.md loading and version hashing |
| `heartbeat.py` | Scheduled proactive agent turns |
| `compaction.py` | Session compression via LLM summarization |
| `delivery.py` | Output callbacks (on_partial_reply, on_block_reply, on_tool_call) |
| `resource_loader.py` | Workspace context file loading |
| `my_mono/` | Core agent primitives: AgentSession, streaming, tool execution |

### Streaming / Block Chunking

Text output is split into blocks by `_BlockChunker`: buffers tokens until ≥ min chars and a flush pattern (sentence boundary) is reached, then fires `on_block_reply`. End of turn fires `on_block_reply_flush`.

### Configuration (`.selma/selma.json`)

Key settings: `model` (provider/model-id), `channels` (telegram, webchat), `heartbeat` (every, target, light_context, isolated_session), `session.reset` (at_hour, idle_minutes), `tools_allow` (all or list), `memory` (vector_search, embed_model).

### Tracing

OpenTelemetry via `my_mono.tracing`. Decorate key async functions with `@tracer.chain()` or `@tracer.agent()`. Phoenix collector on `localhost:6006` (no-op if unavailable).

## Conventions

- All Python comments, docstrings, and Pydantic Field descriptions must be in English.
- Use `uv run <script>.py` to run scripts, never `python` or `uv run python`.
- Async-first: all agent operations use `asyncio`.
- Pydantic v2 `BaseModel` with `model_config` for serialization settings.
- Tool filtering: `tools_allow` in config controls which tools are injected per session; filter happens at runtime, not at tool creation.
- Bootstrap mode: if `BOOTSTRAP.md` has content, the first agent turn uses it as a special setup prompt, then clears the file.

---
> Source: [gkvoelkl/python-selma](https://github.com/gkvoelkl/python-selma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
