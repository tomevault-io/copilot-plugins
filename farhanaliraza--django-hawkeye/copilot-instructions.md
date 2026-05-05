## django-hawkeye

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

django-hawkeye is a Django library for BM25 full-text search using PostgreSQL's pg_textsearch extension. It provides a lightweight Elasticsearch alternative with simple API integration.

## Development Commands

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run all tests (requires PostgreSQL with pg_textsearch)
pytest

# Run a single test file
pytest tests/unit/test_indexes.py

# Run a specific test
pytest tests/unit/test_indexes.py::test_name -v

# Run with coverage
pytest --cov=django_hawkeye

# Linting
ruff check src/
black src/ --check
```

## Test Database Setup

Tests require PostgreSQL 17+ with pg_textsearch extension. Configure via environment variables:
- `POSTGRES_DB` (default: django_hawkeye_test)
- `POSTGRES_USER` (default: postgres)
- `POSTGRES_PASSWORD` (default: postgres)
- `POSTGRES_HOST` (default: localhost)
- `POSTGRES_PORT` (default: 5432)

## Architecture

### Core Components

- **`BM25Searchable`** (`mixins.py`): Mixin that adds `.search(query)` classmethod to models. Auto-discovers BM25Index from model's Meta.indexes.

- **`BM25Index`** (`indexes.py`): Custom Django Index subclass for BM25 indexes. Generates `CREATE INDEX ... USING bm25 ...` SQL with configurable `text_config`, `k1`, and `b` parameters.

- **`BM25SearchQuerySet`** (`search.py`): QuerySet-like wrapper returned by `.search()`. Provides chainable interface (filter, exclude, select_related, etc.) with automatic BM25 scoring annotation.

- **Expressions** (`expressions.py`):
  - `BM25Score`: Annotation expression using `<@>` operator for scoring
  - `BM25Query`: Generates `to_bm25query()` SQL function calls
  - `BM25Match`: Filter expression for threshold matching

- **Migration Operations** (`operations.py`):
  - `CreateExtension`: Generic PostgreSQL extension creator
  - `CreatePgTextsearchExtension`: Specialized for pg_textsearch
  - `CreateBM25Index`: Manual index creation in migrations

### Key Implementation Details

**Score semantics**: pg_textsearch returns NEGATIVE scores. Lower values = better match. Use `.order_by('bm25_score')` (ascending), not `.order_by('-bm25_score')`.

**Index auto-creation**: BM25Index.create_sql() automatically runs `CREATE EXTENSION IF NOT EXISTS pg_textsearch` before creating the index.

## Project Structure

```
src/django_hawkeye/    # Main package
tests/
  unit/                # Unit tests (no DB required for some)
  integration/         # Integration tests (require PostgreSQL)
  models.py            # Test models (Article with BM25Index)
  settings.py          # Django test settings
  conftest.py          # Pytest fixtures
```

---
> Source: [FarhanAliRaza/django-hawkeye](https://github.com/FarhanAliRaza/django-hawkeye) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
