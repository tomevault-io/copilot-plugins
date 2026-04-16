## vectorbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vector Bot is a fully local RAG (Retrieval-Augmented Generation) pipeline using LlamaIndex with Ollama. It indexes documents and answers natural language questions about them, running entirely offline after installation. Supports multiple independent document stores.

## Essential Commands

```bash
# Install for development
pip install -e ".[dev]"

# Store management
vector-bot store new work-docs --docs ./documents
vector-bot store list
vector-bot store default work-docs
vector-bot store info work-docs
vector-bot store reindex work-docs

# Core operations
vector-bot doctor                    # Check Ollama + list stores
vector-bot ingest                    # Index current store
vector-bot query "..."               # Query current store
vector-bot -s work-docs query "..."  # Query specific store

# Migration from legacy index_storage
vector-bot migrate --name my-docs

# Testing
pytest tests/ -v                     # All tests (223 tests)
pytest tests/unit/ -v                # Unit tests only
pytest tests/unit/test_store.py -v   # Single test file
pytest tests/ -k "test_name"         # Run specific test by name

# Quality checks
ruff check src/ tests/
mypy src/

# Build executable
python build_executable.py
```

## Architecture

```
src/rag/
├── cli.py          # Entry point: doctor, ingest, query, store, migrate commands
├── store.py        # Multi-store management (create, list, delete, etc.)
├── config.py       # Configuration loading (global + per-store)
├── migrate.py      # Legacy index_storage migration
├── ingest.py       # Document loading and vector index creation
├── query.py        # Query engine over persisted index
└── ollama_check.py # Ollama server health checks and model detection
```

### Storage Layout

```
~/.vector-bot/                    # Global config home
├── config.json                   # Default store, Ollama settings
└── stores/
    └── <store-name>/
        ├── store.json            # Store metadata (docs_dir, timestamps)
        └── index/                # LlamaIndex persisted storage
```

### Data Flow

1. **Store Resolution** (`store.py`): Resolves `-s` flag or default store
2. **Configuration** (`config.py`): Merges global config with store-specific paths
3. **Doctor** (`ollama_check.py`): Validates Ollama server, lists stores
4. **Ingest** (`ingest.py`): Loads docs from store's `docs_dir`, persists index to store's `index/`
5. **Query** (`query.py`): Loads store's index, queries with LLM, returns response

### Key Dependencies

- **llama-index-core**: Vector index and query engine
- **llama-index-llms-ollama**: Ollama LLM integration
- **llama-index-embeddings-ollama**: Ollama embedding model integration
- **rich**: Console output formatting

### Store Module API

```python
from rag.store import (
    create_store,      # create_store("name", docs_dir=Path(...))
    list_stores,       # Returns list of store info dicts
    get_store,         # get_store("name") -> dict or None
    delete_store,      # delete_store("name") -> bool
    rename_store,      # rename_store("old", "new")
    resolve_store,     # resolve_store(name_or_none) -> resolved name
    set_default_store, # set_default_store("name")
    get_default_store, # Returns default store name or None
)
```

## Testing Notes

- All 223 tests are fully mocked (no Ollama required)
- Test markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.requires_ollama`
- Fixtures defined in `tests/conftest.py`
- Store tests in `tests/unit/test_store.py` (87 tests)

## Build & Distribution

- PyPI package: `pip install vector-bot`
- npm wrapper: `npm install -g @joshuaramirez/vector-bot`
- Standalone executable: Built via PyInstaller (`build_executable.py`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JoshuaRamirez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
