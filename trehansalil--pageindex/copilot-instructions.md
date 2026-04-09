## pageindex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

PageIndex MCP Server — a [FastMCP](https://github.com/jlowin/fastmcp)-based server that exposes document processing capabilities (vectorless reasoning-based RAG) via the Model Context Protocol. Documents are indexed into a hierarchical tree structure and stored in MinIO object storage.

## Environment Setup

Uses `uv` for dependency management (Python 3.12 required):

```bash
uv sync                  # install all dependencies
uv sync --extra dev      # include pytest/httpx for testing
```

Environment variables (copy to `.env`):
- `OPENAI_API_KEY` or `CHATGPT_API_KEY` — required by the PageIndex library
- `OPENAI_BASE_URL` — optional base URL for OpenAI-compatible API providers (e.g. Azure, local models)
- `PAGEINDEX_MODEL` — main LLM model (default: `gpt-4o-2024-11-20`)
- `PAGEINDEX_FILTER_MODEL` — model for pre-filtering documents (default: `gpt-4o-mini`)
- `PAGEINDEX_SEARCH_MODEL` — model for tree search (default: `gpt-4o-mini`)
- `PAGEINDEX_SEARCH_CONCURRENCY` — parallel search concurrency (default: `3`)
- `MINIO_ENDPOINT` — defaults to `10.43.246.106:9000`
- `MINIO_ACCESS_KEY` / `MINIO_SECRET_KEY` — defaults to `minioadmin`/`minioadmin`
- `MINIO_BUCKET` — defaults to `pageindex`
- `MINIO_SECURE` — defaults to `false`
- `REDIS_URL` — Redis connection string (defaults to `redis://neonatal-care-redis.neonatal-care:6379/1`)
- `UPLOAD_API_KEY` — required for the `/upload` endpoint
- `WEB_CONCURRENCY` — number of gunicorn workers (default: `1`; must stay 1 because MCP sessions are in-memory per worker — scale via pod replicas + Traefik sticky sessions)
- `CACHE_TTL` — Redis cache TTL in seconds for processed documents (default: `300`)

## Running the Server

```bash
# Development (single process)
uv run python mcp_server.py

# Production (gunicorn with uvicorn workers)
uv run gunicorn -c gunicorn.conf.py pageindex_mcp.server:app

# Start arq workers (separate process for document processing)
uv run arq pageindex_mcp.worker.WorkerSettings
```

## Uploading Documents

```bash
# PDFs — single file, URL, or folder
uv run python upload.py /path/to/document.pdf
uv run python upload.py https://example.com/report.pdf
uv run python upload.py /path/to/folder/ --workers 4

# Office/image files — converts to PDF via LibreOffice/Pillow first
uv run python convert_and_upload.py /path/to/file.docx
uv run python convert_and_upload.py /path/to/folder/ --convert-only

# Process files from doc_store/ (with hash-based change detection)
uv run python preprocess_client.py                # all files in doc_store/
uv run python preprocess_client.py HR_FAQ.docx    # single file
uv run python preprocess_client.py --bg           # background, logs to preprocess.log
```

## Architecture

**`mcp_server.py`** / **`server.py`** — the core server. Exposes a module-level ASGI `app` for gunicorn and registers these MCP query tools via `FastMCP`:
- `recent_documents()` / `find_relevant_documents(query)` / `get_document(doc_id)` — query the stored index
- `get_document_structure(doc_id)` / `get_page_content(doc_id, pages)` — retrieve document structure and content

**`worker.py`** — arq worker process. Runs `process_document_job` tasks enqueued by the upload endpoint. Start separately from the MCP server so document processing doesn't compete with query serving.

**`cache.py`** — Redis-backed document cache shared across gunicorn workers. `load_doc` checks Redis before hitting MinIO. Invalidated on `save_doc`/`delete_doc`.

**Storage layout in MinIO:**
- `processed/<doc_id>.json` — the indexed tree (title, summary, nodes hierarchy)
- `processed/<doc_id>.meta.json` — lightweight metadata sidecar (doc_id, doc_name, source_url, processed_at) used by listing to avoid downloading full trees
- `uploads/<doc_id>/<filename>` — the raw source file
- `preloaded/<filename>` — source files synced from local `doc_store/`
- `hashes/processed_hashes.json` — change-detection cache used by `preprocess_client.py`

**`preprocess_client.py`** — idempotent batch processor: reads files from `doc_store/`, computes SHA-256, skips unchanged files (comparing against the MinIO hash cache), then calls `upload_and_process_document` via the FastMCP `Client`.

**`upload.py`** / **`convert_and_upload.py`** — CLI helpers that call `process_document` via `langchain_mcp_adapters.MultiServerMCPClient`. `convert_and_upload.py` additionally runs LibreOffice headless for office formats and Pillow for images before uploading.

**`test.py`** — a LangChain ReAct agent example that connects to the running server and queries it with an LLM.

**Key dependency:** `pageindex` is installed from a private GitHub repo (`trehansalil/PageIndex-salil`). The `page_index_main` function does the heavy lifting of PDF parsing and hierarchical indexing; `md_to_tree` handles markdown/text inputs.

## Notes

- `doc_id` values are 8-character UUID prefixes generated at processing time.
- Document processing is offloaded to arq workers via Redis queue — the MCP server only handles queries.
- The upload endpoint (`POST /upload/files`) enqueues jobs; poll `GET /upload/status/{job_id}` for results.
- The server path is hardcoded to port `8201` in all client scripts.
- Production deployments should run gunicorn (multi-worker) + separate arq worker processes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trehansalil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/trehansalil)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
