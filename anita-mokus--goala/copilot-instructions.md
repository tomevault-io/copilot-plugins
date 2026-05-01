## goala

> Core project context for Goala - always apply to every session


# Goala — Project Overview

Goala is a **Dockerized Python RAG (Retrieval-Augmented Generation) system** designed as a general-purpose AI assistant platform for any company. It replaces or augments the work of employees by ingesting company-specific documents and answering questions based on embedded vector search. Each deployment is configured per-company via environment variables and prompt customization.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| API | FastAPI + Uvicorn |
| RAG Framework | LangChain 0.3.x |
| Embeddings | `BAAI/bge-m3` via HuggingFace (CPU-only, singleton) |
| Vector Store | PostgreSQL + pgvector (`langchain-postgres`) |
| Document Parsing | `unstructured[pdf]` library |
| LLM | Configurable provider (Groq / DeepSeek / OpenRouter / Ollama) |
| Database | PostgreSQL 16 with pgvector extension |
| Containerization | Docker multi-stage build + Docker Compose |

## Directory Structure

```
Goala/
├── src/
│   ├── api/main.py          # FastAPI app — /chat, /health, / endpoints
│   ├── core/config.py       # All configuration (env vars, defaults, RAG prompt)
│   ├── services/
│   │   ├── rag_service.py   # RAG pipeline: retrieve + generate
│   │   ├── ingest_service.py# Document ingestion (PDF/TXT/DOCX/HTML/CSV)
│   │   ├── embeddings.py    # HuggingFace embedding singleton
│   │   └── llm_provider.py  # LLM factory (Groq/DeepSeek/OpenRouter/Ollama)
│   └── utils/
│       ├── query_cli.py     # Interactive CLI for querying
│       ├── ingest_cli.py    # CLI for document ingestion
│       └── evaluate_rag.py  # RAG evaluation with LLM-as-judge
├── data/                    # Input documents (mounted read-only in Docker)
├── shared/                  # Evaluation datasets (JSON)
├── docker/init-db.sql       # Enables pgvector extension on DB init
├── Dockerfile               # Multi-stage build (python:3.11-slim)
├── docker-compose.yml       # Services: db (pgvector/pgvector:pg16) + api
├── requirements.txt         # Pinned dependencies
└── .env.example             # Environment variable template
```

## API Endpoints

- `GET /` — status
- `GET /health` — health check (validates DB connection)
- `POST /chat` — `{"message": "string"}` → `{"response": "string"}`

## Key Behaviours

- On startup, the API **auto-ingests all documents** in `data/` if the vector DB collection is empty.
- The embedding singleton (`embeddings.py`) must not be instantiated multiple times — always use the module-level getter.
- LangChain version is **pinned to 0.3.x** — do not suggest upgrades to 1.x.
- The system prompt and assistant persona are defined in `src/core/config.py` and are **configurable per deployment** — each company customizes the prompt, language, and persona to fit their use case.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Anita-Mokus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
