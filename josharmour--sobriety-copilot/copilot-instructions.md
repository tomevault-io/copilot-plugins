## sobriety-copilot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**sobriety-copilot** is a recovery-based multimodal assistant that helps addicts/alcoholics via a RAG-powered chat interface backed by locally-hosted Gemma 4.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the server (Flask on port 5000)
python -m src.server

# Index documents into ChromaDB (place PDFs/TXT/MD in documents/)
curl -X POST http://localhost:5000/api/index
```

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `LLM_BASE_URL` | `http://localhost:11434/v1` | OpenAI-compatible API endpoint (Ollama default) |
| `LLM_MODEL` | `gemma4` | Model name passed to the API |
| `LLM_API_KEY` | `ollama` | API key (Ollama ignores this) |
| `DOCUMENTS_DIR` | `documents` | Directory containing literature to index |
| `RAG_DB_PATH` | `rag_db` | ChromaDB persistent storage path |

## Architecture

The RAG pipeline mirrors [12-step-literature](https://github.com/josharmour/12-step-literature):

1. **Document Processing** (`src/rag/document_processor.py`) — reads PDF (pdfplumber with PyPDF2 fallback), TXT, MD files
2. **Semantic Chunking** (`src/rag/semantic_chunker.py`) — splits text at topic boundaries using sentence-transformer cosine similarity (50–300 word chunks)
3. **Indexing** (`src/rag/indexer.py`) — embeds chunks with `all-MiniLM-L6-v2` and stores in ChromaDB with cosine distance
4. **Retrieval** (`src/rag/retriever.py`) — encodes query, retrieves top-k chunks, formats context with source attribution
5. **Inference** (`src/inference/engine.py`) — sends system + RAG prompt to Gemma 4 via OpenAI-compatible API (Ollama)
6. **Server** (`src/server.py`) — Flask app serving the chat UI and `/api/chat`, `/api/index` endpoints

## Domain Considerations

- This application serves a vulnerable population. Responses must be empathetic, non-judgmental, and privacy-respecting.
- The system prompt directs the LLM to recommend SAMHSA (1-800-662-4357) or 911 for crisis situations.
- User data in this domain is highly sensitive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/josharmour) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
