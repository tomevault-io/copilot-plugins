## mosaico

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Mosaico** is a modular local Document QA & RAG (Retrieval-Augmented Generation) platform. It processes uploaded documents, generates embeddings, and answers questions via a local LLM — no cloud dependencies.

```
Browser → Django Frontend (:9001) → FastAPI AI API (:9000)
                                          ├→ Qdrant (:6333)   — Vector DB
                                          ├→ Ollama (:11434)  — Local LLM
                                          └→ PostgreSQL (:5433) — Persistence
```

## Running the Stack

```bash
make up      # Ollama esterno (OLLAMA_URL dal .env)
make local   # Ollama locale Docker + pull automatico del modello
make down    # Ferma tutto
make logs    # Log in tempo reale
```

Equivalenti senza `make`:
```bash
# Ollama esterno
docker compose up -d --build

# Ollama locale (sovrascrive OLLAMA_URL inline)
OLLAMA_URL=http://ollama:11434/api/generate docker compose --profile local up -d --build
```

Con `--profile local` il servizio `ollama-pull` scarica automaticamente `OLLAMA_MODEL` al primo avvio; i successivi sono no-op grazie al volume `ollama_data`.

Per cambiare modello: aggiornare `OLLAMA_MODEL` in `.env`, poi:
```bash
docker compose exec ollama ollama pull <nuovo-modello>
make local
```

Individual services also have their own `docker-compose.yml` files under `ai_api/`, `mosaico/`, `qdrant_project/`, `ollama_project/`, `postgres_project/`.

## Development (without Docker)

**AI API (FastAPI) — port 9000:**
```bash
cd ai_api
pip install -r requirements.txt
python app.py
# or: uvicorn app:app --host 0.0.0.0 --port 9000 --workers 4
```

**Frontend (Django) — port 9001:**
```bash
cd mosaico
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 0.0.0.0:9001
```

## Testing

```bash
cd ai_api
pytest -q
pytest -q tests/test_app.py -k upload   # Run a specific test
```

## Architecture

### ai_api/
- `app.py` — Monolithic FastAPI app (~2,200 lines): all endpoints, RAG pipeline, document ingestion, chat logic, Qdrant operations, Ollama calls.
- `db.py` — SQLAlchemy models and DB setup (supports SQLite for dev, PostgreSQL for production via `DB_ENGINE` env var).
- `tests/test_app.py` — Pytest test suite.
- `debug/` — Standalone debug scripts for upload, chat, and API.
- `migrate_sqlite_to_postgres.py` — One-shot migration tool.

**RAG flow (5 phases):**
1. **Ingest** — text extraction (pdfplumber, python-docx, openpyxl); supports `.txt`, `.md`, `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.json`
2. **Chunk** — sentence-aware chunking (default 600 chars, overlap 120); respects sentence boundaries
3. **Embed & index** — `sentence-transformers/all-MiniLM-L6-v2` → Qdrant collection `{username}_{collection}`
4. **Retrieve** — multi-vector search → hybrid reranking (60% vector + 25% fuzzy + 15% keyword) → MMR diversity → deduplication → stitching
5. **Generate** — Ollama LLM via streaming SSE

Multi-tenant isolation is achieved via Qdrant **collections** (one per namespace/user group).

### mosaico/ (Django frontend)
- `mosaico/settings.py` — Django config: CORS, JWT (60 min lifetime), logging, static files.
- `ui/views.py` — All page views: login, register, upload, chat, collection config, public chat.
- `ui/templates/` — Bootstrap HTML templates.
- JWT tokens are obtained from `/api/token/` and forwarded by Django views to the FastAPI backend on every request.

### Configuration (root `.env`)
Key variables:
- `OLLAMA_URL`, `OLLAMA_MODEL` — LLM connection
- `DB_ENGINE` (`sqlite` | `postgres`), `PG_*` — Database
- `QDRANT_HOST`, `QDRANT_PORT` — Vector DB
- `CHUNK_SIZE` (default 600), `CHUNK_OVERLAP` (default 120), `EMBEDDING_MODEL` — Ingestion tuning
- `CHAT_RESULT_LIMIT` (10), `CHAT_CANDIDATES` (50), `CHAT_CONTEXT_CHAR_BUDGET` (9000), `CHAT_HISTORY_CHAR_BUDGET` (3000) — Retrieval tuning
- `QDRANT_SCORE_THRESHOLD` (0.42), `MMR_LAMBDA` (0.65) — Similarity/diversity thresholds
- `ENABLE_RERANK`, `ENABLE_MMR`, `ENABLE_STITCH`, `ENABLE_MULTI_VECTOR_SEARCH` — Feature flags (all default `true`)
- `ENABLE_CROSS_ENCODER_RERANK` (default `false`), `ENABLE_RAG_DEBUG` (default `true`) — Optional features
- `ENABLE_SOURCE_LABELS` (default `false`) — Prefix each context excerpt with a `[Fonte: <document>]` label (mitigates source-attribution failures identified in the paper's judge-validation study)
- `SKIP_AUTH` — Bypass JWT auth (development only)
- `DJANGO_SECRET_KEY`, `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS` — Django security
- `API_BASE`, `API_PUBLIC_BASE` — Django → FastAPI connection URLs

## Key API Endpoints (FastAPI)

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/upload` | Ingest document (supports `.txt`, `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.json`) |
| POST | `/chat` | RAG chat (full response) |
| POST | `/chat/stream` | RAG chat — streaming SSE (`data: {"chunk":"..."}` … `data: {"done":true,"conversation_id":"..."}`) |
| GET | `/uploads` | Upload history |
| GET | `/conversations` | Conversation list |
| GET | `/conversations/{id}` | Conversation detail |
| DELETE | `/conversations/{id}` | Delete conversation |
| GET/PUT/DELETE | `/collection/config` | Collection settings |
| GET | `/ollama/models` | List available Ollama models |
| GET | `/healthz` | Full system health check |
| GET | `/docs` | Swagger UI |

## Django Pages (port 9001)

| Path | Purpose |
|------|---------|
| `/` | Home / Dashboard |
| `/login/` | Authentication |
| `/upload/` | Document upload (2-panel: config + dropzone) |
| `/chat/` | RAG chat with streaming |
| `/uploads/` | Upload history with pagination |
| `/collection-config/` | Collection scope/prompt customization |
| `/public-chat/` | Anonymous chat (public mode) |

---
> Source: [angelostefani/Mosaico](https://github.com/angelostefani/Mosaico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
