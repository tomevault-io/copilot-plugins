## af-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 작업 규칙 (Claude Code가 반드시 따를 것)

- **모든 주석은 한글**로 작성한다. 비전공자(프론트엔드 개발자)도 이해할 수 있도록 쉬운 말로 설명한다.

## Project Overview

AF Agent is a FastAPI-based AI assistant for place recommendations (restaurants, accommodations) using RAG (Retrieval-Augmented Generation), LangGraph agents, and Kakao Map data.

## Development Commands

```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # then fill in API keys

# Run locally (dev mode with auto-reload)
python main.py
# or: uvicorn main:app --reload

# Docker (full stack)
docker-compose -f docker-compose.infra.yml up -d  # Start pgvector, etc.
docker-compose up -d                               # Start app
docker-compose logs -f app

# API docs
# Swagger: http://localhost:8000/docs
# ReDoc:   http://localhost:8000/redoc
# Health:  http://localhost:8000/health
```

No test framework or linter is currently configured.

## Architecture

```
FastAPI Routes (routers/)
    └── Service Layer (services/)
            ├── Repositories (repositories/) → PostgreSQL via SQLAlchemy
            ├── Agents (agents/)             → LangGraph ReAct agents
            ├── Crawlers (crawlers/)         → Kakao Map API + Playwright
            └── Memory (memory/)            → pgvector-backed vector store
```

### Key Data Flow

**Chat Message Flow:**
1. POST `/api/chats/{id}/messages` → saves user message to PostgreSQL
2. RAG service: query → OpenAI embedding → pgvector similarity search → context
3. Claude (via OpenRouter) generates response with context + chat history
4. AI response saved to PostgreSQL and returned

**Data Pipeline Flow:**
1. POST `/pipeline/run` → BackgroundTask
2. Kakao API keyword search → Playwright browser scraping → OpenAI embedding → pgvector upsert
3. Poll status at GET `/pipeline/status`

### Configuration (`src/config.py`)

All settings are loaded from `.env` via Pydantic Settings. Key variables:
- `RAG_MODEL` — LLM for generation (default: `anthropic/claude-sonnet-4.5` via OpenRouter)
- `EMBEDDING_MODEL` — OpenAI embedding model (default: `text-embedding-3-small`)
- `PGVECTOR_DATABASE_URL` — Vector DB connection string
- `RAG_TOP_K` — Number of retrieved context items (default: 5)
- `ENABLE_PIPELINE_ROUTES` — Toggle pipeline endpoints
- PostgreSQL connection: `PG_HOST`, `PG_USER`, `PG_PASSWORD`, `PG_DATABASE`, `PG_PORT`

### Authentication

JWT-based (7-day expiration, bcrypt password hashing). Protected endpoints require `Authorization: Bearer {token}` header. Use `get_current_user` dependency from `src/utils/auth.py`.

### External Services

| Service | Purpose |
|---------|---------|
| PostgreSQL | User/Chat/Message persistence (SQLAlchemy ORM) |
| pgvector | Vector store for place embeddings (`kakao_places` table) |
| OpenAI API | Embeddings (`text-embedding-3-small`) |
| OpenRouter | LLM inference (Claude model) |
| Kakao API | Place search + Playwright browser scraping |

### Code Structure Conventions

- `routers/` — FastAPI route handlers only (thin layer)
- `services/` — Business logic (auth, chat, RAG, pipeline)
- `repositories/` — Database CRUD operations
- `schemas/` — Pydantic request/response models
- `agents/` — LangGraph-based agent definitions
- `crawlers/` — Kakao Map API client and Playwright scraper

## CI/CD

GitHub Actions (`.github/workflows/first.yml`): on push/PR to `main`, builds Docker image and pushes to GHCR (`ghcr.io/{owner}/af_agent`), then deploys to EC2 via SSH. Required secrets: `SSH_HOST`, `SSH_USER`, `SSH_PRIVATE_KEY`, and all API keys from `.env`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MC-agent) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
