## claude-like-ask-user-question

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A Human-in-the-Loop (HITL) demo that implements Claude-style `ask_user_question` using LangGraph's `interrupt()` mechanism. Exposed as an OpenAI-compatible API consumed by OpenWebUI, with an injected JavaScript widget that renders interactive option cards.

## Commands

```bash
# Install dependencies (creates .venv + installs)
uv sync

# Start backend (listens on 0.0.0.0:8888)
uv run python src/web_server.py

# Start OpenWebUI + nginx (renders nginx config, starts/reloads both services)
./start_frontend.sh

# Override ports
NGINX_PORT=3001 OPENWEBUI_PORT=8080 BACKEND_PORT=8888 ./start_frontend.sh
```

Configuration via `.env` (copy from `.env.example`): `LLM_API_KEY` is required; `LLM_BASE_URL`, `LLM_MODEL`, and `API_PORT` are optional.

## Architecture

```
Browser → nginx (:3001) → OpenWebUI (:8080)
              │                  │
              │ sub_filter       │ /v1/chat/completions
              │ injects JS       ↓
              │             FastAPI Backend (:8888)
              │             LangGraph ReAct Agent
              │             → ask_user_question tool
              │             → interrupt()
              │                  │
              └── ask_question.js served directly by nginx
                 (frontend JS scans code blocks, renders option cards)
                 (user click → postMessage → OpenWebUI sends message)
                 (next API call detects pending interrupt → Command(resume=...))
```

### Key design decisions

- **Tool → frontend bridge**: The `ask_user_question` tool outputs a JSON payload inside a fenced code block (` ```ask-user-question`) in the SSE stream. The frontend JS (`frontend/ask_question.js`) scans the DOM for these blocks and replaces them with interactive widgets.
- **Resume detection**: `POST /v1/chat/completions` checks `has_pending_interrupt()` on the agent state. If an interrupt is pending, the latest user message is treated as a `Command(resume=...)`; otherwise it starts a fresh turn.
- **Resume guard**: `StreamManager.is_resuming` flag prevents the `ask_user_question` tool from re-emitting the question payload during resume — it goes straight to `interrupt()` to return the answer.
- **Session identity**: session ID = `sha256(user_id:chat_id)[:32]`, tying OpenWebUI's user+chat to a LangGraph thread.
- **State storage**: In-memory only — `MemorySaver` for LangGraph checkpoints, `SessionStore` (OrderedDict LRU, max 100 entries) for agent instances.
- **Demo agent**: A rock-paper-scissors game (`src/agent.py`) that demonstrates the HITL flow — the agent picks randomly, asks the user for their move via `ask_user_question`, then judges the outcome.

### Core modules

| File | Role |
|---|---|
| `src/web_server.py` | FastAPI app, SSE streaming, session lifecycle, auto-resume logic |
| `src/agent.py` | Builds the LangGraph ReAct agent with RPS system prompt |
| `src/tools/ask_user_question.py` | Tool factory — emits code-fenced JSON, calls `interrupt()`, returns user reply |
| `src/utils/agent_runner.py` | Runs `astream_events`, dispatches tool/model events to stream manager, checks interrupt state |
| `src/infra/stream_manager.py` | Thread-safe SSE chunk routing (callback-per-thread), `is_resuming` flag |
| `src/infra/checkpointer.py` | Singleton `MemorySaver` |
| `src/config.py` | Frozen dataclass config from env vars |
| `frontend/ask_question.js` | Self-contained IIFE widget — DOM scanning, widget rendering, `postMessage` submit |
| `nginx/nginx.conf.template` | Self-contained nginx config template — rendered by `start_frontend.sh` via envsubst |

---
> Source: [sudoplayer/Claude-like-ask-user-question](https://github.com/sudoplayer/Claude-like-ask-user-question) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
