## dooers-agents-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**agents-server** is a Python SDK for building AI agent agents that communicate via WebSocket. It provides real-time message handling, conversation persistence, analytics tracking, and configurable settings for agent services deployed on the dooers.ai platform.

## Development Commands

```bash
# Setup (uses uv for dependency management)
uv sync --group dev

# Code Quality (via poethepoet)
poe format              # Format code with ruff
poe format:imports      # Sort imports
poe check               # Run ruff linter
poe check:fix           # Auto-fix lint issues

# Testing
poe test                # Run all tests (pytest)
poe test:cov            # Run with coverage

# Dependency Updates
poe pcu                 # Check for updates
poe pcu:upgrade         # Upgrade dependencies
```

## Architecture

### Core Components

```
src/dooers_agents/
├── server.py           # AgentServer: main entry point, manages connections
├── config.py           # AgentConfig: database and feature configuration
├── dispatch.py         # DispatchStream: programmatic handler execution
├── repository.py       # Repository: direct DB access for threads/events
├── registry.py         # ConnectionRegistry: tracks active WebSocket connections
├── broadcast.py        # BroadcastManager: push events to subscribers
├── handlers/
│   ├── router.py       # Frame routing and agent handler execution
│   ├── pipeline.py     # HandlerPipeline: shared setup/execute for WS and dispatch
│   ├── on.py           # AgentOn: incoming message context
│   ├── send.py         # AgentSend: factory for yielding response events
│   └── memory.py       # AgentMemory: conversation history access
├── protocol/
│   ├── models.py       # Thread, ThreadEvent, Run, ContentPart models
│   ├── frames.py       # WebSocket frame definitions (C2S/S2C)
│   └── parser.py       # Frame serialization/deserialization
├── persistence/
│   ├── base.py         # Persistence protocol (abstract interface)
│   ├── sqlite.py       # SQLite implementation
│   └── postgres.py     # PostgreSQL implementation (asyncpg)
└── features/
    ├── analytics/      # Event tracking and feedback collection
    └── settings/       # Configurable agent settings schema
```

### Handler Pattern

Agent handlers are async generators that receive context and yield response events:

```python
async def agent_handler(on, send, memory, analytics, settings):
    yield send.run_start()
    history = await memory.get_history(limit=20, format="openai")
    # ... LLM call ...
    yield send.text("response text")
    yield send.run_end()
```

### WebSocket Protocol

- **Client-to-Server (C2S)**: Connect, ThreadList, ThreadSubscribe, EventCreate, Feedback, SettingsPatch
- **Server-to-Client (S2C)**: Ack, ThreadSnapshot, EventAppend, RunUpsert, SettingsSnapshot

### Database Support

- **SQLite**: Default for development (`database_type="sqlite"`)
- **PostgreSQL**: Production with SSL support, managed databases (AlloyDB, Cloud SQL)

Database connection configured via environment variables: `AGENT_DATABASE_HOST`, `AGENT_DATABASE_PORT`, `AGENT_DATABASE_USER`, `AGENT_DATABASE_NAME`, `AGENT_DATABASE_PASSWORD`, `AGENT_DATABASE_SSL`

## Key Patterns

- All models use **Pydantic v2** for validation and serialization
- Database operations use **SQLAlchemy 2.0** async patterns
- WebSocket protocol is frame-based with typed payloads
- Thread/event hierarchy: Agent → Thread → ThreadEvent (with optional Run grouping)
- Multi-format history: `memory.get_history(format="openai|anthropic|google|cohere|voyage")`

---
> Source: [Dooers-ai/dooers-agents-server](https://github.com/Dooers-ai/dooers-agents-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
