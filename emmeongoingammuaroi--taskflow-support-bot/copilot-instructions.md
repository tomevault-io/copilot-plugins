## taskflow-support-bot

> AI-powered customer support bot for "TaskFlow", a SaaS project management tool. Built with RAG pipeline, LangGraph agent, HITL workflow, domain-driven design, and production-grade API hardening (JWT auth, rate limiting, structured error handling, health/readiness checks, CI).

# TaskFlow Support Bot

## Project Overview

AI-powered customer support bot for "TaskFlow", a SaaS project management tool. Built with RAG pipeline, LangGraph agent, HITL workflow, domain-driven design, and production-grade API hardening (JWT auth, rate limiting, structured error handling, health/readiness checks, CI).

**Stack:** Python 3.12 + FastAPI | React + TypeScript + Vite | OpenAI API | Qdrant | MongoDB | LangGraph | MCP

**Run:** `docker compose up` → 4 services (backend, web, mongo, qdrant)

---

## Architecture Rules

### Layered Responsibility — STRICT

```
domain/  → Business rules only. Deterministic. No I/O, no LLM calls. 100% unit testable.
tools/   → Side effects + I/O. Calls domain/ internally for validation.
agent/   → LangGraph reasoning. Binds tools/ directly (in-process, no MCP hop).
mcp.py   → Thin adapter. Wraps tools/ for external protocol interop. Never add logic here.
rag/     → Retrieval pipeline. Chunker, embedder, reranker, HyDE. No business rules.
```

**"LLM proposes, domain validates"** — LLM suggests priority/category, `domain/ticket_rules.py` overrides with deterministic ITIL matrix. Business rules NEVER live in prompts.

### What Goes Where

- **Thread, TranscriptEntry, Block types, AgentEvent** → `api/models.py` (conversation infra, NOT domain)
- **Ticket, KBArticle** → `api/domain/entities.py` (business entities WITH behavior)
- **Priority matrix, SLA computation, ticket ID generation** → `api/domain/ticket_rules.py`
- **Escalation logic** → `api/domain/escalation.py`
- **DB access** → `api/db.py` (Motor async, no repository pattern)
- **Config/env vars** → `api/config.py` (single Pydantic Settings class)
- **FastAPI dependencies** → `api/deps.py`
- **User account (auth)** → `api/models.py` (User model — conversation/account infra, NOT domain)
- **JWT + password hashing** → `api/core/security.py`
- **Rate limiting** → `api/core/rate_limit.py`
- **Custom exceptions + global handlers** → `api/core/exceptions.py`

### Do NOT

- Create abstract interfaces or repository pattern (1 DB = no abstraction needed)
- Create agent registry (1 agent = direct instantiation)
- Hand-roll RAG plumbing (chunking, embedding, retrieval helpers) — use LangChain/standard libs. This project's depth is the agent/domain/HITL layer, not RAG internals
- Use FastAPI-Users or other heavy auth frameworks — hand-rolled JWT (`python-jose` + `passlib`) is enough for a single-role (customer) auth model. MongoDB access here uses Motor directly (no ODM); FastAPI-Users' MongoDB support requires Beanie, which would pull in an ODM inconsistent with the rest of this codebase
- Add Redis/Celery just for rate limiting — use in-process `slowapi` (in-memory limiter); this project has no background-job workload that would justify a queue
- Add microservices for tools (each tool is ~30 LOC, all in-process)
- Use Zustand or global state in frontend (hooks are sufficient for 1-page app)
- Use EventSource for SSE (use fetch + ReadableStream to support POST body)

---

## Coding Conventions

### Python (api/)

- **Async everywhere** — FastAPI + Motor + async OpenAI client
- **Type hints** on all function signatures
- **Pydantic models** for all data structures (BaseModel or dataclass where appropriate)
- **No classes for services** — use plain async functions. Only domain entities get classes.
- **Imports:** absolute from project root (`from api.domain.ticket_rules import validate_priority`)
- **Error handling:** raise HTTPException in routers, return Result-style in domain/tools
- **Tests:** pytest + pytest-asyncio, files named `test_*.py` in `api/tests/`

### TypeScript (web/)

- **React functional components** only, no class components
- **Custom hooks** for all state logic (`use-chat.ts`, `use-threads.ts`)
- **shadcn/ui** for primitives, Tailwind for styling
- **No barrel exports** (no `index.ts` re-exports)
- **Types** in `lib/types.ts`, API calls in `lib/api.ts`

### Docker

- Python image: `python:3.12-slim`
- Node image: `node:20-slim`
- All env vars in `.env`, loaded via docker-compose `env_file`
- Healthchecks on all services

---

## Key Patterns

### Auth

- Hand-rolled JWT (no FastAPI-Users): `api/core/security.py` — `hash_password()`, `verify_password()`, `create_access_token()`, `decode_access_token()`
- `User(id, email, hashed_password, created_at)` lives in `api/models.py`
- `POST /auth/register`, `POST /auth/login` in `api/routers/auth.py`
- `get_current_user` dependency in `api/deps.py` — extracts + validates JWT from `Authorization: Bearer` header
- Every Thread has a `user_id`; thread/chat routes filter by the requesting user (no cross-user access), return 404 (not 403) for another user's thread to avoid leaking existence

### Rate Limiting

- `slowapi` with in-memory storage (no Redis) — `api/core/rate_limit.py`
- Applied per-user (fallback per-IP for unauthenticated routes) on `/threads/{id}/chat` and `/auth/*`
- Limit via `RATE_LIMIT_PER_MINUTE`; exceeding it returns 429 with a structured error body

### Error Handling

- Custom exception classes in `api/core/exceptions.py` (e.g. `NotFoundError`, `UnauthorizedError`, `ValidationError`)
- Global exception handlers registered in `api/app.py` — every error response follows `{"error": {"code": str, "message": str}}`
- Routers/domain/tools raise the custom exceptions; handlers translate to the right HTTP status — no bare `except Exception` swallowing errors silently

### Health Checks

- `GET /health` — liveness, always 200 once the process is up
- `GET /ready` — readiness, pings MongoDB (`db.command("ping")`) and Qdrant (`get_collections()`); 503 if either is unreachable
- Docker Compose healthchecks call these endpoints

### HITL (Human-in-the-Loop)

"Read freely, write carefully":
- `kb_retrieve`, `keyword_search` → auto-allow
- `create_ticket` → interrupt, user must confirm
- LangGraph `interrupt()` → checkpoint to MongoDB → resume on user action

### RAG Pipeline

```
query → [HyDE expand] → embed → Qdrant top-20 → [rerank] → top-5 → generate
```

- HyDE and reranker are toggleable via `HYDE_ENABLED` / `RERANKER_ENABLED`
- Chunker: LangChain `RecursiveCharacterTextSplitter`, chunk_size=512, overlap=50
- Embedder: LangChain `OpenAIEmbeddings` (text-embedding-3-small, 1536d), batch up to 20
- Reranker: LangChain `CrossEncoderReranker` wrapping `cross-encoder/ms-marco-MiniLM-L-6-v2` (sentence-transformers), lazy-loaded
- RAG components use LangChain/standard libraries throughout — no hand-rolled retrieval internals here (see [Do NOT])

### SSE Streaming

Backend yields `AgentEvent` discriminated union → SSE → frontend parses:
- `text_delta` — streaming text token
- `tool_call` — agent calling a tool
- `tool_result` — tool execution result
- `interrupt` — HITL pause, awaiting user input
- `done` — stream complete

---

## Environment Variables

```
OPENAI_API_KEY=
OPENAI_MODEL=gpt-4o-mini
EMBEDDING_MODEL=text-embedding-3-small
MONGODB_URL=mongodb://mongodb:27017
QDRANT_URL=http://qdrant:6333
QDRANT_COLLECTION=kb_articles
HYDE_ENABLED=true
RERANKER_ENABLED=true
OBSERVABILITY_PROVIDER=none
LANGFUSE_PUBLIC_KEY=
LANGFUSE_SECRET_KEY=
JWT_SECRET_KEY=
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=1440
RATE_LIMIT_PER_MINUTE=60
```

---

## Project Structure

```
api/
  routers/    — FastAPI route handlers (incl. auth.py)
  core/       — security (JWT/hashing), rate_limit, exceptions
  domain/     — deterministic business rules (no I/O, no LLM)
  agent/      — LangGraph graph, HITL, runner
  rag/        — chunker, embedder, reranker, HyDE, ingest (LangChain-based)
  tools/      — kb, search, ticket (plain async functions)
  eval/       — metrics, dataset, runner
  tests/      — pytest
web/src/
  hooks/      — use-chat, use-threads
  components/ — chat UI components
  lib/        — types, api, sse
data/
  articles/   — KB markdown files
  eval/       — test queries
scripts/      — seed, eval CLI
.github/
  workflows/  — ci.yml (lint + test on push/PR)
```

---
> Source: [emmeongoingammuaroi/taskflow-support-bot](https://github.com/emmeongoingammuaroi/taskflow-support-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
