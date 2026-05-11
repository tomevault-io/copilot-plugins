## king-context

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

King Context is a local-first, token-efficient documentation server for LLM context injection via the Model Context Protocol (MCP). It uses a 4-layer cascade search strategy (Cache → Metadata → FTS5 → Hybrid embeddings) that stops at the first hit, reducing token usage by 59-69% compared to cloud alternatives like Context7.

## Rules

- All code, comments, variable names, function names, and documentation must be written in English
- Git commits and PR descriptions must be in English
- Architecture docs and tasks can be in Portuguese
- Never mix languages in the same file

## Commands

```bash
# Install
pip install -e .

# Run MCP server
king-context                         # via console script
python -m king_context.server        # via module

# Run tests
pytest
pytest tests/test_db.py -v           # verbose database tests
pytest -k <test_name>                # run specific test

# Seed database (indexes all data/*.json into SQLite)
python -m king_context.seed_data

# Reset and reseed database
./scripts/run-seed-data.sh

# Scraper — scrape and index documentation
king-scrape <url>                    # full pipeline (discover→filter→fetch→chunk→enrich→export)
king-scrape <url> --yes              # skip enrichment confirmation prompt
king-scrape <url> --stop-after fetch # run up to fetch, then stop
king-scrape <url> --step export      # resume from a specific step
king-scrape <url> --no-llm-filter    # disable LLM filter (heuristic only)
```

## Architecture

**Core package: `src/king_context/`**

- **server.py** — FastMCP server entry point. Exposes 4 MCP tools: `search_docs`, `list_docs`, `show_context`, `add_doc`. Loads embedding model and data at startup, then delegates to `db.py`. Entry point: `main()`.
- **db.py** — Search engine and database layer. Implements `search_cascade()` with 4 layers: `_check_cache()` → `_search_metadata()` → `_search_fts()` → `_rerank_with_embeddings()`. All SQLite/FTS5 operations live here. Module-level state holds the embedding model, numpy array, and section mapping.
- **seed_data.py** — Data loading. `seed_all()` indexes all `data/*.json`; `seed_one(path)` indexes a single file.
- **`__init__.py`** — Defines `PROJECT_ROOT = Path.cwd()` used by all modules to resolve paths relative to CWD.

**Database (SQLite + FTS5) — `docs.db` at project root:**

- `documentations` — doc metadata (name is UNIQUE)
- `sections` — content with JSON-serialized keywords/use_cases/tags fields and integer priority
- `sections_fts` — FTS5 virtual table (external content from sections) for BM25 full-text search
- `query_cache` — maps normalized queries to section IDs for <1ms cache hits

**Embeddings:**

- Model: `all-MiniLM-L6-v2` (SentenceTransformer), lazy-loaded in `server.py`
- `data/embeddings.npy` — numpy float32 array, one row per section
- `data/_internal/section_mapping.json` — maps section DB IDs to embedding array indices
- Hybrid layer uses cosine similarity with 0.3 threshold; gracefully skipped if files missing

**Search transparency:** Every search response includes a `transparency` object with `method`, `latency_ms`, `search_path`, and `from_cache` fields.

## Documentation JSON Schema

Files in `data/*.json` follow this structure — the metadata fields (`keywords`, `use_cases`, `tags`, `priority`) power the fast metadata search layer:

```json
{
  "name": "api-name",
  "display_name": "Display Name",
  "version": "v1",
  "base_url": "https://docs.example.com",
  "sections": [{
    "title": "Section Title",
    "path": "section-path",
    "url": "https://docs.example.com/section",
    "keywords": ["keyword1", "keyword2"],
    "use_cases": ["how to do X", "when to use Y"],
    "tags": ["category1", "category2"],
    "priority": 10,
    "content": "Markdown content..."
  }]
}
```

## Testing Patterns

- Tests live in `tests/` with shared fixtures in `tests/conftest.py`
- Tests use `tmp_path` fixture for isolated temporary databases
- `monkeypatch` overrides `db.DB_PATH` to point at temp databases
- Server tool tests mock `search_cascade()` and other `db` functions
- Patch targets use full module paths: `king_context.db.xxx`, `king_context.server.xxx`
- Test files: `tests/test_db.py` (cascade/FTS/cache/metadata/schema), `tests/test_server.py` (MCP tools), `tests/test_seed_data.py` (data loading), `tests/test_embeddings.py` (dependency versions), `tests/test_load_embeddings.py` (embedding persistence)

## Path Resolution

All paths are resolved from `CWD` (the project where king-context is installed):

- `.king-context/docs/` — Indexed documentation store (`STORE_DIR`)
- `.king-context/data/` — Raw JSON doc files (scraper exports here)
- `.king-context/_temp/` — Scraper work directories (`TEMP_DOCS_DIR`)
- `.king-context/_learned/` — Agent self-learning shortcuts
- `.king-context/core/venv/` — Python virtual environment (managed by installer)
- `.king-context/bin/` — CLI wrapper scripts (kctx, king-scrape)

## Installer (`installer/`)

npm package `@king-context/cli` that sets up king-context in any project:

```bash
npx @king-context/cli init      # Install everything
npx @king-context/cli doctor    # Check installation health
npx @king-context/cli update    # Update tools + skills (preserves data)
```

- Zero npm dependencies — pure Node.js built-ins
- Creates venv, installs Python package, copies skills, configures Claude Code
- Templates in `installer/templates/` (skills, wrappers, env, claude-md snippet)

## Key Directories

- `src/king_context/` — Core Python package (server, db, seed_data, scraper)
- `src/context_cli/` — CLI package (kctx command)
- `installer/` — npm installer package (@king-context/cli)
- `tests/` — All test files and conftest.py
- `scripts/` — Shell scripts (run-seed-data.sh)
- `data/` — Indexed documentation JSONs and embedding files (legacy, repo-local)
- `.claude/skills/` — Claude Code skills (king-context, scraper-workflow)

---
> Source: [deandevz/king-context](https://github.com/deandevz/king-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
