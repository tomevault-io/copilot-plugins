## toolref

> ToolRef is a production-grade RAG engine that turns professional documentation into on-demand domain knowledge for AI Agents (via MCP) and humans (via Chat UI). Status: Pre-alpha.

# CLAUDE.md — ToolRef

## Project Overview
ToolRef is a production-grade RAG engine that turns professional documentation into on-demand domain knowledge for AI Agents (via MCP) and humans (via Chat UI). Status: Pre-alpha.

## Architecture
- **Backend**: FastAPI + LangGraph agentic RAG engine (Python, uv)
- **Frontend**: React 18 + TypeScript + Shadcn/ui + Vite
- **Infrastructure**: Milvus (vectors), PostgreSQL (metadata), Redis (semantic cache)
- **MCP Server**: Exposes `rag_query`, `document_add`, `namespace_list`
- **Embeddings**: BGE-M3 (1024-dim, dense + sparse)
- **Reranking**: BGE-reranker-v2-m3

## Directory Structure
```
backend/           Python backend (FastAPI + LangGraph)
  app/             Main application package
    api/           REST API routes
    cache/         Semantic cache (Redis)
    db/            Database models + Alembic migrations
    ingestion/     Document parsing pipeline
    mcp/           MCP server implementation
    retrieval/     LangGraph RAG engine
    storage/       File storage
    vectorstore/   Milvus vector operations
  tests/           pytest tests
  alembic/         Migration scripts
frontend/          React frontend
  src/
    api/           API client
    components/    UI components
    pages/         Page components
    types/         TypeScript types
scripts/           Dev utilities (seed, env setup, health check)
docs/              Documentation
```

## Development Commands
```bash
make up              # Start all services (Docker Compose)
make down            # Stop all services
make logs            # Tail all logs
make lint            # Run ruff linter
make format          # Run ruff formatter
make test            # Run pytest
make migrate         # Run Alembic migrations
make clean           # Remove volumes and orphan containers
```

## Key Conventions
- Backend uses `uv` for Python dependency management
- Frontend uses `npm`
- All infrastructure runs via Docker Compose
- Namespace isolation: each knowledge domain is a separate namespace
- LLM dev: Ollama + Qwen2.5 local; prod: DeepSeek-V3 / GPT-4o (swappable)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiveden)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hiveden)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
