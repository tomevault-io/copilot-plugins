## notion-py

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unofficial Python 3 client for Notion.so's internal API (v3). Provides an object-oriented interface that maps Notion database tables to Python classes with automatic format conversion and local caching.

**Authentication**: Uses `token_v2` cookie from a logged-in Notion browser session.

## Commands

```bash
# Install
pip install -e .

# Build package
make build        # sdist + wheel

# Release to PyPI
make release      # build + twine upload

# Run smoke tests (requires live Notion credentials)
python run_smoke_test.py --page [NOTION_PAGE_URL] --token [NOTION_TOKEN_V2]
# Or set NOTION_TOKEN env var instead of --token
```

There is no unit test suite — only an integration smoke test against a live Notion page.

## Architecture

### Data Flow

```
NotionClient (client.py)
  ├── HTTP Session (requests + retry logic for 429/502/503/504)
  ├── RecordStore (store.py) — central thread-safe cache
  │     Records are stored as: _values[table][id] = record_data
  └── Record instances (Block, Collection, User, Space)
        └── Always read/write through RecordStore, never hold state locally
```

### Core Classes

- **`Record`** (`records.py`): Base class for all Notion objects. Provides `get()`/`set()`/`refresh()` and a callback system for reactive updates. All subclasses reference data in the central `RecordStore` rather than storing state internally.

- **`Block`** (`block.py`): Represents any Notion content block. ~30+ subclasses (PageBlock, TextBlock, TodoBlock, CollectionViewBlock, etc.) registered in the `BLOCK_TYPES` dict. `Children` class manages parent-child relationships.

- **`Collection` / `CollectionView`** (`collection.py`): Database tables and their views. `CollectionRowBlock` rows auto-generate getter/setter attributes from the collection schema (slugified column names). Query building via `build_query()`/`default_query()`.

- **`RecordStore`** (`store.py`): Thread-safe central data cache with mutex. Optional disk caching (disabled by default). Manages callbacks and triggers them on data changes using `dictdiffer`.

- **`NotionClient`** (`client.py`): Main entry point. Manages HTTP session, RecordStore, authentication, and transaction submission. Key methods: `get_block()`, `get_collection()`, `get_collection_view()`.

### Property Mapping System (`maps.py`)

Custom descriptors that bidirectionally map between Python attributes and Notion's internal data structures:

- `field_map(path, python_type)` — maps block fields with type conversion
- `property_map(name, python_type)` — maps collection row properties with automatic markdown conversion
- `joint_map(main, aliases)` — combines multiple mappings under one name

### Markdown Conversion (`markdown.py`)

Converts between Notion's internal rich text format (nested arrays with annotations) and CommonMark markdown. Handles bold, italic, code, strikethrough, links, and LaTeX.

### Operations & Transactions (`operations.py`)

`build_operation()` constructs API operations for Notion's `submitTransaction` endpoint. `NotionClient.submit_transaction()` sends them. Use `as_atomic_transaction()` context manager for batching.

### Monitor (`monitor.py`)

Long-polling subscription for real-time updates via Primus protocol. **Currently broken and disabled by default** — use `record.refresh()` for manual updates.

## Key Patterns

- All record types proxy data through `RecordStore` — never cache state on instances
- Block type dispatch uses `BLOCK_TYPES` dict mapping type strings to classes
- Collection row properties are dynamically created from schema as slugified attribute names
- Callbacks run in daemon threads to avoid blocking
- API retries: up to 10 retries with exponential backoff
- `set()` retries up to 20 times with 0.1s delay to handle server-side eventual consistency

## Environment Variables

- `NOTION_TOKEN` — default auth token for smoke tests
- `NOTIONPY_LOG_LEVEL` — logging level: debug, info, warning, error, disabled (logs to `~/.notion-py/notion.log`)

---
> Source: [jamalex/notion-py](https://github.com/jamalex/notion-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
