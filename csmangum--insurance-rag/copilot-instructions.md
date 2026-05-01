## insurance-rag

> Insurance RAG — a multi-domain Retrieval-Augmented Generation system for US insurance industries. It uses a domain-plugin architecture where each insurance line (Medicare, Auto, Property, etc.) registers its own data sources, extractors, enrichment rules, topic definitions, query patterns, and system prompt. Shared infrastructure handles embeddings, vector storage, chunking, hybrid retrieval, and LLM generation.

# AGENTS.md

## Project Overview

Insurance RAG — a multi-domain Retrieval-Augmented Generation system for US insurance industries. It uses a domain-plugin architecture where each insurance line (Medicare, Auto, Property, etc.) registers its own data sources, extractors, enrichment rules, topic definitions, query patterns, and system prompt. Shared infrastructure handles embeddings, vector storage, chunking, hybrid retrieval, and LLM generation.

Currently implemented domains: **Medicare** (fully functional) and **Auto Insurance** (scaffolded).

**Language:** Python 3.11+
**Package manager:** pip / setuptools (see `pyproject.toml`)

## Repository Layout

```
src/insurance_rag/           # Main package (installed as editable via `pip install -e .`)
  __init__.py
  config.py                 # Paths, env config, multi-domain support (ACTIVE_DOMAINS, domain_data_dir, etc.)
  domains/                  # Domain plugin system
    __init__.py              #   Domain registry (register_domain, get_domain, list_domains)
    base.py                  #   InsuranceDomain abstract base class
    medicare/                #   Medicare domain plugin
      __init__.py             #   MedicareDomain class (source kinds: iom, mcd, codes)
      patterns.py             #   LCD patterns, query expansion, synonyms, system prompt
      data/topics.json        #   Medicare clinical/policy topic definitions
    auto/                    #   Auto Insurance domain plugin
      __init__.py             #   AutoInsuranceDomain class (source kinds: regulations, forms, claims, rates)
      patterns.py             #   Coverage patterns, query expansion, synonyms, system prompt
      states.py               #   State-specific config (tort system, min liability, PIP, top markets)
      data/topics.json        #   Auto insurance topic definitions
  download/                 # Phase 1: download data (Medicare-specific downloaders)
    __init__.py              #   Re-exports download_iom, download_mcd, download_codes
    iom.py                   #   IOM chapter PDF scraper
    mcd.py                   #   MCD bulk ZIP downloader
    codes.py                 #   HCPCS + ICD-10-CM code file downloader
    _manifest.py             #   Manifest writing and SHA-256 hashing
    _utils.py                #   URL sanitization, stream_download helper
  ingest/                   # Phase 2: text extraction, enrichment, chunking, clustering, summarization
    __init__.py              #   SourceKind type
    extract.py               #   PDF/text extraction (pdfplumber, optional unstructured fallback)
    enrich.py                #   HCPCS/ICD-10 semantic enrichment
    chunk.py                 #   LangChain text splitters; optional summary generation
    cluster.py               #   Topic clustering (loads topic defs from active domain)
    summarize.py             #   Extractive summarization (TF-IDF sentence scoring)
  index/                    # Phase 3: embedding and vector store
    __init__.py              #   Re-exports get_embeddings, get_or_create_chroma, upsert_documents
    embed.py                 #   sentence-transformers embeddings
    store.py                 #   ChromaDB upsert (collection_name from domain); get_raw_collection helper
  query/                    # Phase 4: retrieval and RAG chain
    __init__.py
    retriever.py             #   Domain-aware retriever with query expansion and topic-summary boosting
    expand.py                #   Cross-source query expansion (patterns loaded from active domain)
    hybrid.py                #   HybridRetriever: semantic + BM25 via RRF with cross-source diversification
    chain.py                 #   Local LLM RAG chain (system prompt from domain)
app.py                      # Streamlit UI with domain selector (launch: `streamlit run app.py`)
scripts/                    # CLI entry points (all support --domain flag)
  download_all.py            #   Bulk download (--domain medicare|auto|all, --source, --force)
  ingest_all.py              #   Extract, chunk, embed, store (--domain, --source, --force, --skip-*)
  validate_and_eval.py       #   Index validation + retrieval eval (hit rate, MRR)
  query.py                   #   Interactive RAG REPL (--domain, --filter-source, --filter-state)
  run_rag_eval.py            #   Full-RAG eval report generation
  eval_questions.json        #   Eval question set (expected keywords/sources)
tests/                      # Unit tests (pytest; install with pip install -e ".[dev]")
  conftest.py                #   Shared fixtures (autouse: reset BM25 index after each test)
  test_domains.py            #   Domain registry, interface compliance, domain-specific tests
  test_config.py             #   Safe env int/float parsing
  test_download.py           #   Mocked HTTP, idempotency, zip-slip and URL sanitization
  test_ingest.py             #   Extraction and chunking tests
  test_enrich.py             #   HCPCS/ICD-10-CM semantic enrichment tests
  test_cluster.py            #   Topic clustering tests
  test_summarize.py          #   Extractive summarization tests
  test_index.py              #   Chroma/embedding tests
  test_query.py              #   Retriever and chain tests
  test_retriever_boost.py    #   Summary document boosting and topic injection
  test_hybrid.py             #   Hybrid retriever, BM25, RRF, source diversity
  test_search_validation.py  #   Search/validation tests
  test_app.py                #   Streamlit app helpers (requires .[ui])
data/                       # Runtime data directory (gitignored)
  medicare/                  #   Medicare domain data
    raw/                      #   Downloaded Medicare files
    processed/                #   Extracted/chunked Medicare text
  auto/                      #   Auto Insurance domain data
    raw/                      #   Downloaded auto insurance files
    processed/                #   Extracted/chunked auto text
  chroma/                    #   ChromaDB (separate collections per domain)
```

## Setup

1. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```
2. Install the package in editable mode:
   ```bash
   pip install -e .
   ```
3. Copy `.env.example` to `.env` for optional configuration overrides. No API keys are required — embeddings and LLM inference run locally via sentence-transformers and HuggingFace.

## Running Tests

Always use a virtual environment. Install the dev optional dependency (includes pytest and rank-bm25), then run:

```bash
pip install -e ".[dev]"
pytest tests/ -v
```

Tests use `unittest.mock` to mock HTTP calls and external dependencies. No network access or real data downloads are needed. Some index tests are skipped automatically when ChromaDB is unavailable. A shared `conftest.py` fixture resets the BM25 singleton index after each test for isolation.

## Code Style and Quality

- Formatter/linter: **ruff** (install via `pip install -e ".[dev]"`)
- Ruff config: `target-version = "py311"`, `line-length = 100`, rules `E, F, W, I, B, UP`
- No backwards compatibility constraints — refactor freely as needed
- Type hints are used throughout; follow existing patterns
- All source code lives under `src/insurance_rag/` with the `pythonpath` set in `pyproject.toml`

## Domain Plugin Architecture

The system uses a **domain-plugin architecture** where each insurance line is a self-contained domain.

### Adding a New Domain

1. Create `src/insurance_rag/domains/<name>/__init__.py` with an `@register_domain` class inheriting from `InsuranceDomain`
2. Implement all abstract methods (see `domains/base.py`)
3. Add `download.py`, `extract.py`, `patterns.py`, `data/topics.json` as needed
4. Add state config if applicable (e.g., `states.py`)
5. Run `python scripts/download_all.py --domain <name>`
6. Run `python scripts/ingest_all.py --domain <name>`

### InsuranceDomain Interface

Each domain provides:

| Concern         | Method                         | Description                                    |
| --------------- | ------------------------------ | ---------------------------------------------- |
| Identity        | `name`, `display_name`         | Short ID and human-readable label              |
| Storage         | `collection_name`              | Unique ChromaDB collection per domain          |
| Data sources    | `source_kinds`                 | List of source type identifiers                |
| Download        | `get_downloaders()`            | Map of source-kind -> download callable        |
| Extraction      | `get_extractors()`             | Map of source-kind -> extract callable         |
| Enrichment      | `get_enricher()`               | Optional semantic enrichment                   |
| Topics          | `get_topic_definitions_path()` | Path to `topics.json` for clustering           |
| Query detection | `get_query_patterns()`         | Regex patterns for domain-specific queries     |
| Expansion       | `get_source_patterns()`        | Source relevance detection patterns            |
| Synonyms        | `get_synonym_map()`            | Domain-specific synonym expansion              |
| Prompt          | `get_system_prompt()`          | LLM system prompt tailored to the domain       |
| States          | `get_states()`                 | US state codes (None for federal domains)      |
| Chunking        | `get_chunk_overrides()`        | Per-source-kind chunk size overrides           |
| UI              | `get_quick_questions()`        | Example questions for the Streamlit UI         |

### Domain Registry

Domains auto-register via `@register_domain` decorator when their module is imported. Discovery happens lazily via `_discover_domains()` which imports built-in domain packages.

```python
from insurance_rag.domains import get_domain, list_domains

list_domains()  # -> ["auto", "medicare"]
domain = get_domain("medicare")
domain.collection_name  # -> "medicare"
```

## Key Conventions

- **Configuration** is centralized in `src/insurance_rag/config.py`. Key additions for multi-domain:
  - `ACTIVE_DOMAINS` — list of enabled domain names (env-configurable)
  - `DEFAULT_DOMAIN` — domain used when none is specified
  - `domain_data_dir(name)`, `domain_raw_dir(name)`, `domain_processed_dir(name)` — domain-partitioned data paths
- **Idempotent operations**: downloads check for existing manifests/files before re-downloading. Index upserts are incremental by content hash. Use `--force` to override.
- **Separate collections**: each domain gets its own ChromaDB collection (e.g., `"medicare"`, `"auto_insurance"`)
- **No API keys**: the system uses local sentence-transformers for embeddings and a local HuggingFace model (default: TinyLlama) for generation.

## Important Patterns

- HTTP clients use `httpx` with context managers and streaming for large downloads
- ZIP extraction uses safe extractors that guard against zip-slip path traversal
- The vector store (ChromaDB) persists at `data/chroma/` with per-domain collection names
- `get_or_create_chroma(embeddings, collection_name=...)` accepts an explicit collection name
- Query expansion patterns, synonyms, and system prompts are loaded from the active domain at runtime
- Topic definitions for clustering are loaded from the active domain's `topics.json`, falling back to `DATA_DIR/topic_definitions.json` or the package default
- The hybrid retriever (`query/hybrid.py`) maintains a module-level singleton `BM25Index`; use `reset_bm25_index()` in tests

## Retrieval Architecture

The retrieval pipeline has two retriever implementations selected by `get_retriever()`:

1. **`HybridRetriever`** (default when `rank-bm25` is installed via `.[dev]` or `.[hybrid]`):
   - Expands queries into source-targeted variants via `query.expand` (domain-configurable patterns)
   - Runs semantic search (Chroma) and BM25 keyword search for each variant
   - Merges results via **Reciprocal Rank Fusion** (RRF) with configurable weights
   - Applies **cross-source diversification** to guarantee minimum representation from each relevant source type
   - Domain-specific specialized queries trigger additional source-filtered searches

2. **`LCDAwareRetriever`** (fallback when `rank-bm25` is unavailable):
   - For specialized queries: runs multi-variant source-filtered searches + base search, deduplicates via round-robin
   - For general queries: standard similarity search

Both retrievers apply **topic summary boosting**: `detect_query_topics` identifies relevant topics, `inject_topic_summaries` fetches anchor docs, and `boost_summaries` promotes them.

## Optional Dependencies

Defined in `pyproject.toml` under `[project.optional-dependencies]`:

- **`dev`**: `ruff`, `pytest`, `rank-bm25` — for linting, testing, and hybrid retrieval
- **`ui`**: `streamlit` — for the embedding search UI (`app.py`)
- **`hybrid`**: `rank-bm25` — for hybrid (semantic + keyword) retrieval only
- **`unstructured`**: `unstructured` — PDF fallback for scanned/image PDFs

---
> Source: [csmangum/insurance_rag](https://github.com/csmangum/insurance_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
