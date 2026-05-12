## opencode-memory-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **@csuwl/opencode-memory-plugin** - a native OpenCode plugin that provides persistent memory with semantic search capabilities. It uses the @opencode-ai/plugin API for seamless integration.

## Development Commands

```bash
# Install dependencies
cd opencode-memory-plugin && npm install

# Run installation script manually
node bin/install.cjs

# Create a tarball for testing
npm pack

# Install from local tarball
npm install -g ./csuwl-opencode-memory-plugin-*.tgz
```

## Architecture

The plugin consists of three main layers:

### 1. Tool Layer (`plugin.js`)
Exports 12 tools via the @opencode-ai/plugin API:
- **Memory**: `memory_write`, `memory_read`, `memory_search`, `vector_memory_search`
- **Daily Logs**: `init_daily`, `list_daily`
- **Sessions**: `save_session`, `list_sessions`
- **Index Management**: `rebuild_index`, `update_index`, `configure_index`, `index_status`

All tool execute functions must return JSON.stringify() for successful results.

### 2. Search Layer (`lib/`)
- **vector-store.js**: Manages embeddings via @huggingface/transformers, stores vectors in sqlite-vec, provides hybrid search (vector + BM25)
- **bm25.js**: Pure JavaScript BM25 implementation for keyword-based search
- **index-manager.js**: Handles incremental index updates with file hash detection, debouncing, and batch processing

### 3. Installation Layer (`bin/install.cjs`)
Automatic setup that:
- Creates `~/.opencode/memory/` directory structure
- Copies 9 core memory files (SOUL.md, AGENTS.md, USER.md, etc.)
- Creates `memory-config.json` with v2.0 defaults
- Updates OpenCode's `opencode.json` to register the plugin

## Key Dependencies

- **@opencode-ai/plugin**: Native OpenCode plugin API
- **@huggingface/transformers**: Local embedding model inference
- **better-sqlite3**: SQLite database for vector storage
- **sqlite-vec**: Vector similarity search extension

## Memory Files Structure

```
~/.opencode/memory/
├── MEMORY.md          # Long-term memory
├── SOUL.md            # Personality & boundaries
├── AGENTS.md          # Operating instructions
├── USER.md            # User profile
├── IDENTITY.md        # Assistant identity
├── TOOLS.md           # Tool conventions
├── HEARTBEAT.md       # Health checklist
├── BOOT.md            # Startup checklist
├── BOOTSTRAP.md       # One-time ritual
├── daily/             # Daily logs
├── sessions/          # Session records
├── .index-hashes.json # Change tracking
├── vector-index.db    # SQLite vector store
└── memory-config.json # Configuration
```

## Configuration

Config file: `~/.opencode/memory/memory-config.json`

Key settings:
- `search.mode`: "hybrid" (default), "vector", "bm25", or "hash"
- `embedding.model`: Default is "Xenova/all-MiniLM-L6-v2" (384 dimensions, 80MB)
- `embedding.enabled`: Set false to disable vector search
- `indexing.chunkSize`/`chunkOverlap`: Text chunking parameters

## Embedding Models

| Model | Dimensions | Size | Best For |
|-------|------------|------|----------|
| Xenova/all-MiniLM-L6-v2 | 384 | 80MB | Baseline (default) |
| Xenova/bge-small-en-v1.5 | 384 | 130MB | Best balance |
| Xenova/bge-base-en-v1.5 | 768 | 400MB | Maximum quality |

When switching models, run `rebuild_index force=true` to reindex with correct dimensions.

## Search Behavior

**Hybrid Search** (default): `score = 0.7 * vector_score + 0.3 * bm25_score`

The plugin automatically falls back to BM25 keyword search when:
- Embedding model fails to load
- `embedding.enabled` is false
- Model initialization errors occur

## Debugging

The vector store creates `vector-index.db` in the memory directory. You can inspect it:

```bash
sqlite3 ~/.opencode/memory/vector-index.db "SELECT COUNT(*) FROM documents;"
```

For embedding model issues, check:
- Network connectivity (first-time model download)
- `~/.cache/huggingface/` for cached models
- Set `HF_HUB_URL` environment variable for HuggingFace mirrors

---
> Source: [csuwl/opencode-memory-plugin](https://github.com/csuwl/opencode-memory-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
