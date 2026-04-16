## agent-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup
uv sync
uv sync --extra dev

# Run (dev)
uv run uvicorn agent_platform.main:app --reload --port 8000

# Local infra
docker compose up -d

# Apply migrations (run in order)
psql $DATABASE_URL -f migrations/001_init_users.sql
psql $DATABASE_URL -f migrations/002_init_sessions.sql
psql $DATABASE_URL -f migrations/003_init_quotas.sql

# Tests
uv run pytest
uv run pytest tests/test_core/test_pool.py   # single file
uv run pytest -k "test_allocate"             # single test

# Lint
uv run ruff check src/
uv run ruff format src/
```

## Architecture

Sandbox management platform. External callers (RL engines, eval harnesses) call this platform to create/exec/destroy sandboxes. This platform does not do RL training or eval grading — it only manages sandbox lifecycle.

```
verl/slime (RL)          Eval Harness          Agent Service
      │                       │                      │
 /v1/rl/*             /v1/sandbox/*            /v1/agent/*
      └───────────────────────┼─────────────────────┘
                        Task Dispatcher
                   ┌──────────┼──────────┐
                RL Pool    Eval Pool   Agent Pool
                (ECS)      (ACI)       (ECS)
                   └──────────┼──────────┘
                ┌─────────────┼─────────────┐
           ECSProvider   ACIProvider    FCProvider
```

**Key design decisions:**
- **Gateway hot path** = Redis only. API Key → SHA-256 → Redis cache (TTL 5min, DB on miss). Quota = Redis INCR. Usage = Redis Stream (async flush to DB). Zero DB calls on hot path.
- **RL Pool (ECS)** pre-warmed, supports `reset()` for reuse across trajectories. Soft limit → 429 Retry-After. Hard limit → 429.
- **Eval Pool (ACI)** always fresh containers from caller-specified Docker image, destroyed after use. Never pre-warmed (image varies per task).
- **Session** = one trajectory → multiple sandbox slots by `tool_type`. Supports `attach_sandbox()` mid-trajectory.
- **Dispatcher** sync path (blocks) and async path (task_id → Redis → optional callback POST).

## Key Files

| File | Purpose |
|------|---------|
| `src/agent_platform/api/middleware.py` | Auth + quota (Redis-only hot path) |
| `src/agent_platform/api/v1/sandbox.py` | Sandbox API: `POST /`, `POST /{id}/exec`, `POST /{id}/upload`, `DELETE /{id}` |
| `src/agent_platform/api/v1/rl.py` | RL batch API: `allocate_batch`, `step_batch`, `reset` |
| `src/agent_platform/core/pool.py` | `SandboxPool` + `PoolManager` |
| `src/agent_platform/core/session.py` | `SessionManager` — traj↔sandbox binding |
| `src/agent_platform/core/dispatcher.py` | Sync/async dispatch |
| `src/agent_platform/providers/base.py` | `SandboxProvider` ABC |
| `src/agent_platform/providers/aci.py` | ACI — Docker image → container (Eval Pool) |
| `migrations/` | Raw SQL, apply in order by filename prefix |

## Adding a New Provider

Implement `SandboxProvider` in `src/agent_platform/providers/`, register in `core/pool.py` `PoolManager.start()`.

## API Keys

Stored as SHA-256 hash. Plaintext `sk-...` returned only at creation. Never log plaintext keys.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/merlintang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
