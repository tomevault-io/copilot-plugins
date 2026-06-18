## kbcraft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

`kbcraft` is a Python CLI tool that builds RAG-ready knowledge bases from Markdown files. It scaffolds `.md` files, organizes/validates them, chunks and embeds content, then pushes to vector stores (Chroma, Qdrant, Pinecone, FAISS). The project is in early development — most modules contain only stub docstrings.

The CLI entry point is `kbcraft.cli:main`. Vector store integrations live under `src/kbcraft/vector_stores/` and are installed as optional extras.

## Commands

```bash
# Install dependencies
poetry install                   # base install
poetry install --all-extras      # with all vector store backends

# Run tests
poetry run pytest                # all tests (with coverage by default)
poetry run pytest tests/test_chunker.py  # single test file

# Lint & format
poetry run ruff check src/ tests/
poetry run black src/ tests/
poetry run mypy src/kbcraft

# Validate environment
poetry run python validate_setup.py
```

Make targets are available (`make test`, `make lint`, `make format`, `make clean`) but use bare `pytest`/`ruff`/`black` without `poetry run` — prefer the `poetry run` forms above.

## Code Style

- Line length: 100 (black + ruff)
- Target: Python 3.8+
- Ruff rules: E, F, W only (E501 ignored — handled by black)
- `mypy` is configured but `disallow_untyped_defs = false`

## Architecture

```
src/kbcraft/
  cli.py          # CLI entry point (main())
  scaffold.py     # Generate .md file sets from templates/outlines
  selector.py     # Select files for ingestion (FileFilter, LANGUAGE_PRESETS)
  chunker.py      # Split docs into chunks
  embedder.py     # Generate vector embeddings from chunks
  sync.py         # Incremental re-indexing (diff-based)
  utils.py        # Shared utilities
  vector_stores/  # One module per backend (chroma, qdrant, pinecone, faiss)
tests/            # pytest suite
```

The pipeline flow is: scaffold → organize → chunk → embed → sync to vector store. Each stage maps to its own module. Vector store backends are conditionally imported based on installed extras.

---
> Source: [nikwork/kbcraft](https://github.com/nikwork/kbcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
