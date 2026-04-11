## archivist

> Archivist is a local file indexer and RAG (Retrieval-Augmented Generation) system for macOS. It monitors `~/Downloads` (configurable), extracts content from files, generates embeddings, and provides hybrid search (lexical + semantic) through a web UI.

# Archivist - AI Agent Context

## Project Overview

Archivist is a local file indexer and RAG (Retrieval-Augmented Generation) system for macOS. It monitors `~/Downloads` (configurable), extracts content from files, generates embeddings, and provides hybrid search (lexical + semantic) through a web UI.

## Current Status

- **Phase 1**: File watching and metadata indexing ✓
- **Phase 2**: Content extraction and embeddings ✓
- **Phase 3**: Web UI and hybrid search ✓

## Goals

- Index files automatically as they appear in the watched directory
- Extract text content from various file types (text, PDF, images, spreadsheets)
- Generate embeddings for semantic search
- Provide fast hybrid search combining FTS, trigrams, and vector similarity
- Web UI for searching and managing the index

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3.12+ with strict typing (mypy) |
| Package Manager | uv |
| Database | SQLite with FTS5 |
| Embeddings | Qwen3-Embedding-8B via Ollama |
| Reranking | Qwen3-Reranker-0.6B via Ollama |
| OCR/Caption | Ministral-3:14b via Ollama |
| PDF Parsing | PyMuPDF (fitz) |
| Spreadsheets | pandas + openpyxl |
| Tokenization | tiktoken (cl100k_base) |
| API | FastAPI + Jinja2 templates |
| Frontend | Tailwind CSS + Alpine.js |
| File Watching | watchdog (FSEvents on macOS) |
| Logging | structlog (JSON format) |

## Code Conventions

### Typing
- **Strict mypy**: All code must pass `mypy --strict`
- Use `from __future__ import annotations` for forward references
- Prefer `pathlib.Path` over string paths
- Use `dataclasses` or `pydantic` models for structured data

### Imports
- Use `from __future__ import annotations` at the top
- Use `TYPE_CHECKING` block for type-only imports
- Sort with ruff (isort rules)

### Logging
- Use structlog for all logging
- Include context with `structlog.contextvars.bind_contextvars()`
- Log structured data, not string formatting

### Error Handling
- Use specific exception types
- Log errors with full context before re-raising
- Use `contextlib.suppress()` for expected exceptions

### Async
- Use async/await for I/O operations
- Use `asyncio.gather()` for concurrent operations
- Prefer `aiofiles` for file I/O

## Architecture Decisions

### Database
- SQLite for simplicity and portability (single file)
- Embeddings stored as BLOBs (can upgrade to sqlite-vec later)
- FTS5 for full-text search on filenames and chunk content
- Trigram table for fuzzy matching

### Job Queue
- SQLite-based job queue (no Redis/RabbitMQ)
- Jobs table with status, heartbeat, retry logic
- Three job types: METADATA → CONTENT → EMBEDDING
- Batch processing to optimize model loading

### Model Loading
- Models loaded on-demand, kept warm with `keep_alive`
- Single coordinator manages pipeline
- Embeddings generated in batches (default 32)

### File Processing
- Three-phase: metadata (fast) → content (medium) → embedding (slow)
- SHA256 for deduplication
- Token-aware chunking with overlap for embeddings
- Chunking strategies: TEXT (512 tokens), TABULAR (1500), DOCUMENT (1000)

### Content Processors
- `TextProcessor`: .txt, .md, .json, .log, code files
- `TabularProcessor`: .csv, .tsv, .xlsx, .xls (via pandas)
- `PDFProcessor`: .pdf (via PyMuPDF)
- `ImageProcessor`: .jpg, .png, etc. (via VLM for OCR/caption)

## Key Files

```
src/archivist/
├── config.py           # Settings via pydantic-settings
├── logging_config.py   # structlog configuration
├── __main__.py         # CLI entry point
├── db/
│   ├── connection.py   # SQLite setup
│   ├── schema.sql      # Database schema (v2)
│   ├── migrations.py   # Schema migrations
│   ├── models.py       # Data models (FileRecord, Job, Chunk)
│   └── repositories/   # CRUD operations
│       ├── files.py    # File records
│       ├── jobs.py     # Job queue
│       └── chunks.py   # Chunks and embeddings
├── watcher/            # File system monitoring
│   ├── debouncer.py    # Event debouncing
│   ├── handler.py      # File event handling
│   └── observer.py     # FSEvents observer
├── pipeline/           # Processing orchestration
│   ├── chunking.py     # Token-aware text splitting
│   ├── coordinator.py  # Batch job orchestration
│   ├── queue_manager.py # Job processing loop
│   └── processors/     # File type handlers
│       ├── base.py     # BaseProcessor + registry
│       ├── metadata.py # Metadata extraction
│       ├── text.py     # Text/code files
│       ├── tabular.py  # CSV/Excel
│       ├── pdf.py      # PDF files
│       └── image.py    # Images (VLM)
├── models/             # Ollama model interfaces
│   ├── client.py       # HTTP client with retries
│   ├── embedder.py     # Embedding generation
│   └── vlm.py          # Vision model (OCR/caption)
├── search/             # Search implementations
│   ├── hybrid.py       # Hybrid search engine (FTS + trigram + vector)
│   ├── scorer.py       # Score normalization and RRF fusion
│   └── reranker.py     # Ollama reranker integration
├── api/                # FastAPI application
│   ├── app.py          # Application factory
│   ├── deps.py         # Dependency injection
│   ├── schemas.py      # Pydantic request/response models
│   └── routes/         # API endpoints
│       ├── health.py   # Health check
│       ├── search.py   # Search API
│       └── files.py    # File listing/stats API
└── web/                # Web UI
    ├── templates/      # Jinja2 templates
    │   ├── base.html   # Base template (Tailwind + Alpine.js)
    │   └── index.html  # Search page
    └── static/
        └── app.js      # Alpine.js search component
```

## Running the Project

```bash
# Install dependencies
uv sync --extra dev

# Run tests
uv run pytest

# Run with coverage
uv run pytest --cov=src/archivist

# Type checking
uv run mypy src/

# Linting and formatting
uv run ruff check src/ tests/
uv run ruff format src/ tests/

# Start the application
uv run archivist start

# Reset database
uv run archivist reset --yes

# CLI help
uv run archivist --help
```

## Testing Guidelines

- Write tests alongside implementation (not after)
- Use fixtures from `tests/conftest.py`
- Mock Ollama calls to avoid real model inference
- Use in-memory SQLite for fast database tests
- Target 80%+ code coverage

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ARCHIVIST_WATCH_PATH` | `~/Downloads` | Directory to watch |
| `ARCHIVIST_DATABASE_PATH` | `~/.archivist/archivist.db` | Database file path |
| `ARCHIVIST_LOG_LEVEL` | `INFO` | Logging level |
| `ARCHIVIST_OLLAMA_HOST` | `http://localhost:11434` | Ollama API endpoint |
| `ARCHIVIST_EMBEDDING_MODEL` | `qwen3-embedding:8b` | Embedding model |
| `ARCHIVIST_VLM_MODEL` | `ministral-3:14b` | Vision model |

## Common Tasks

### Adding a New File Processor
1. Create `src/archivist/pipeline/processors/<type>.py`
2. Inherit from `BaseProcessor`
3. Define `extensions` and `mime_types` class variables
4. Implement `process()` method returning `ProcessingResult`
5. Use `@register_processor` decorator
6. Import in `coordinator.py` to register

### Adding a New API Route
1. Create route module in `src/archivist/api/routes/`
2. Use FastAPI router pattern
3. Add to main app in `src/archivist/api/main.py`

## Known Issues / TODO

### Future: Vector Search
- Currently using brute-force cosine similarity in Python
- Should upgrade to sqlite-vec for efficient vector search
- Table is ready (`chunk_embeddings`), just needs extension loading

### Future: Search Improvements
- Query expansion and synonym matching
- Search result caching
- More granular file type filters in UI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mxmarchal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mxmarchal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
