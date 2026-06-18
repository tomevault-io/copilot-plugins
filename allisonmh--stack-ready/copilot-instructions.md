## stack-ready

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup

```bash
# Activate the virtual environment
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run the API server
uvicorn api:app --reload
```

The project requires an `OPENAI_API_KEY` in a `.env` file (loaded via `python-dotenv`).

## Architecture

This is a RAG (Retrieval-Augmented Generation) application built around `study-guide.txt`, a full-stack developer interview study guide. The intended data flow is:

1. **`ingest.py`** — Loads and chunks `study-guide.txt`, embeds it, and stores vectors in ChromaDB (`langchain-chroma`)
2. **`models.py`** — Pydantic models for request/response schemas
3. **`api.py`** — FastAPI app exposing HTTP endpoints for querying
4. **`agent.py`** — LangChain agent that retrieves relevant chunks from Chroma and calls the OpenAI LLM to answer questions
5. **`observability.py`** — Logging/tracing instrumentation

The vector store (ChromaDB) acts as the bridge between ingestion and query time — `ingest.py` writes to it, `agent.py` reads from it.

## Key Dependencies

| Package | Role |
|---|---|
| `fastapi` + `uvicorn` | HTTP API server |
| `langchain` + `langchain-openai` | LLM orchestration and OpenAI integration |
| `langchain-chroma` + `chromadb` | Vector store for document embeddings |
| `openai` | Direct OpenAI SDK access |
| `tiktoken` | Token counting for chunking |
| `pydantic` | Data validation / models |

---
> Source: [AllisonMH/stack-ready](https://github.com/AllisonMH/stack-ready) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
