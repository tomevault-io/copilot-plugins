## chatgpt-sidecar

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sidecar is a local-first context archive with semantic search capabilities. It's a FastAPI application that ingests documents (particularly ChatGPT exports) and provides semantic search using FAISS vector embeddings.

## Development Commands

### Quick Start
```bash
# Setup virtual environment and dependencies
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Run development server
uvicorn app.main:app --host 127.0.0.1 --port 8088 --reload

# Or use the development script
./scripts/run_dev.sh
```

### Common Tasks
- **Run server**: `uvicorn app.main:app --host 127.0.0.1 --port 8088 --reload`
- **Environment setup**: Copy `.env.example` to `.env` and configure values
- **Database initialization**: Automatic on first run via `app/main.py`

## Architecture

### Core Modules

- **app/main.py**: FastAPI application and route definitions. Global FAISS store management, API authentication.
- **app/db.py**: SQLite database operations using context managers. Handles users, documents, chunks, and embeddings.
- **app/vectorstore.py**: FAISS vector store for semantic search. Uses sentence-transformers for embeddings, IndexFlatIP for cosine similarity.
- **app/ingest_chatgpt.py**: Parses and ingests ChatGPT conversation exports. Handles conversation flattening and fingerprinting.
- **app/text.py**: Text chunking utilities with configurable chunk size (default 1200 chars) and overlap (120 chars).

### API Endpoints

- `GET /healthz` - Health check
- `POST /search` - Semantic search (params: query, k, index_name)
- `POST /ingest/chatgpt-export` - Import ChatGPT conversations
- `POST /reindex` - Rebuild FAISS index from database

### Database Schema

Key tables include:
- **user/project**: Multi-tenancy support
- **document**: Document metadata with fingerprinting
- **chunk**: Text chunks for search
- **embedding_ref**: Maps chunks to FAISS index positions
- **conversation/message**: Structured conversation data

### Environment Configuration

Required `.env` variables:
- `API_KEY`: Authentication key (default: "change-me")
- `DB_PATH`: SQLite database location
- `INDEX_PATH`: FAISS index file location
- `EMBED_MODEL`: Sentence transformer model (default: all-MiniLM-L6-v2)

## Development Guidelines

### When modifying the codebase:
1. Maintain context manager pattern in `app/db.py` for database operations
2. Use batch processing for embeddings (batch_size=64 in vectorstore)
3. Preserve fingerprint-based deduplication for documents
4. Keep API authentication via X-API-Key header for all endpoints except /healthz

### Adding new document types:
1. Create new ingestion module similar to `app/ingest_chatgpt.py`
2. Implement document parsing and chunk generation
3. Add corresponding API endpoint in `app/main.py`
4. Update database schema if needed for document-specific metadata

### Vector store operations:
- FAISS uses normalized embeddings for cosine similarity
- Indexes are stored uncompressed (IndexFlatIP) for accuracy
- Always rebuild index via `/reindex` after bulk document changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidkarpay)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davidkarpay)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
