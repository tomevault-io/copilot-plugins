## docsai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DocsAI** — A local-first hybrid RAG (Retrieval-Augmented Generation) search engine. Transforms heterogeneous documents (web pages, PDFs, Markdown, emails, etc.) into a searchable, AI-powered knowledge base with citation-backed answers. Uses profile-based isolation so each knowledge base has its own config, cache, and vector store.

## Commands

### Setup
```bash
python -m venv .venv
.\.venv\Scripts\activate        # Windows
pip install -r requirements.txt
```

### Running
```bash
# Start the API server for a profile (serves on http://127.0.0.1:8080)
python -m docsai.main serve <profile>

# Ingest documents for a profile (CLI mode)
python -m docsai.main ingest <profile>

# Ask a question via CLI
python -m docsai.main ask <profile> "your question"
```

### Testing
Tests are standalone scripts that run against a live server (must start `serve` first on port 8080):
```bash
python tests/test_complete_flow.py           # E2E: profile create → ingest → query
python tests/test_ingestion.py               # Ingestion pipeline
python tests/test_supercharged_prompts.py    # Prompt mode detection
```

### Utilities
```bash
python scripts/monitor_ingestion.py          # Watch ingestion progress in real-time
python scripts/ingestion_status.py           # Check current ingestion state
python scripts/diagnose_rag_quality.py       # Analyze retrieval quality
python scripts/clear_chroma.py               # Wipe vector store for a profile
```

## Architecture

### Hybrid Retrieval Pipeline (`docsai/retriever/search.py`)
Every query runs two parallel ranking strategies, then merges results:
1. **BM25 sparse ranking** — tokenized keyword matching via `rank-bm25`
2. **Dense embedding ranking** — cosine similarity via `sentence-transformers` (BAAI/bge-base-en-v1.5)

Results are de-duplicated and the top `combine_top_k` (default 10) are passed to the LLM.

### Document Processing Pipeline
`file_parsers.py` → `document_intelligence.py` → `ingestion_manager.py` → ChromaDB

1. **FileParser** routes by extension (15+ formats: HTML, PDF, MD, DOCX, email, EPUB, etc.)
2. **DocumentIntelligence** categorizes documents (technical, conversation, correspondence, reference, literature, data, media) and selects a chunk strategy per category (section_aware, conversation_aware, endpoint_aware, etc.)
3. **IngestionManager** (thread-safe singleton) runs ingestion in a background daemon thread with status progression: IDLE → PREPARING → SCANNING → PROCESSING → INDEXING → COMPLETED/FAILED/CANCELLED

### Multi-Mode LLM Prompting
`llm_runner_supercharged.py` auto-detects question intent and selects a prompt persona:
- **comprehensive** (default) — deep expert analysis
- **integration** — step-by-step builder guidance
- **debugging** — problem-solver detective
- **learning** — educational explainer

The `/ask` endpoint auto-detects mode from question keywords. Override with `?mode=<mode>`.

### Profile System
Each profile is a self-contained knowledge base at `profiles/<name>/`:
- `config.yaml` — source config, retrieval params, LLM settings
- `cache/` — crawled HTML and metadata
- `data/chroma/` — persistent ChromaDB vector store

Config is loaded via `config_loader.py` which deep-merges profile YAML over built-in `_DEFAULTS`.

### Citation Flow
Source URLs are stored in ChromaDB metadata (`source_url` key). After LLM generates an answer with `[1]`, `[2]` markers, the system maps citation numbers back to actual URLs via index lookup.

## Key API Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/ask?q=<question>&mode=<mode>` | Query the knowledge base |
| POST | `/profiles/create` | Create a new profile |
| GET | `/profiles/list` | List all profiles |
| POST | `/ingestion/start/{profile}` | Start background ingestion |
| GET | `/ingestion/status/{task_id}` | Poll ingestion progress |
| POST | `/ingestion/cancel/{task_id}` | Cancel running ingestion |
| POST | `/profile/switch/{profile_name}` | Switch active profile |
| GET | `/profile/{profile_name}/stats` | Get profile statistics |

## External Dependencies

- **Ollama** must be running at `http://127.0.0.1:11434` with model `qwen2.5:14b-instruct` (or whatever is configured in profile YAML)
- Alternative: **llama.cpp** with a GGUF model (set `model.llm.mode: llamacpp` and `model.llm.path` / `model.llm.llama_binary` in profile config)
- **ChromaDB** stores vectors on disk — metadata values must be primitive types (str, int, float, bool); lists/dicts are serialized before storage

## Conventions

- Global state (`global_cfg`, `global_profile`) is set by the `serve` CLI command and shared across FastAPI endpoints
- Search results are 3-tuples: `(doc_id, passage_text, metadata_dict)`
- Retrieval parameters are in config under `retrieval.k_bm25`, `retrieval.k_embed`, `retrieval.combine_top_k`
- The `IngestionManager` is a thread-safe singleton — only one ingestion task runs at a time
- Web crawling respects `robots.txt` and uses configurable depth limits

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/end1989) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
