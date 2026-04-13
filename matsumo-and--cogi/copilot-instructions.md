## cogi

> 1. **Local-first**: All features work completely offline

# Cogi - Development Guidelines

## Core Principles

1. **Local-first**: All features work completely offline
2. **Simplicity**: Minimal dependencies, avoid complexity
3. **Performance**: Index 1-20 repos in under 5 minutes
4. **Practicality**: Focus on actual code search and understanding

## Architecture

### SQLite-Centric Design

Everything is stored in SQLite:
- Symbol Index (structured metadata)
- Call/Import Graph
- Ownership Index
- FTS5 (full-text search with auto-sync triggers)
- Embeddings (vectors stored as BLOBs)

**Why SQLite?**
- Zero additional dependencies (fully local)
- Unified metadata and vector storage
- Brute-force cosine similarity is fast enough for 1-20 repos
- No separate processes needed

### Vector Index Granularity

1. **Class/Struct Level**: Overview (class + method signatures + docs)
2. **Function/Method Level**: Details (implementation + signature + docs)

## Critical Build Requirement

**IMPORTANT: Always use the `fts5` build tag**

```bash
# Correct
go build -tags fts5 ./cmd/cogi
make build

# Wrong - FTS5 won't work
go build ./cmd/cogi
```

## Performance Requirements

- Full index build: ≤5 minutes
- Incremental update: seconds to tens of seconds
- Search response: ≤1 second

**Optimizations:**
- Parallel processing (goroutines per file/repo)
- Batch operations (Ollama embeddings, SQLite inserts)
- SQLite: WAL mode, optimized cache_size
- FTS5: Run `PRAGMA optimize` regularly

## Key Implementation Notes

### Data Consistency
- Use transactions properly
- Set up FTS5 auto-sync triggers (INSERT/UPDATE/DELETE)
- Handle file deletions with cleanup
- Use content_hash for real change detection

### Incremental Updates
- Timestamp-based diff detection
- Re-parse only changed files
- Partial SQLite updates (including embeddings)

### Tree-sitter Parsing
- File size limit: 10MB default
- Exclude patterns: node_modules, vendor, etc.
- Skip on parse error with warning

### Ollama Integration
- Batch size: 32 (configurable)
- Graceful degradation: keyword search works even if semantic search fails
- Clear error messages when Ollama is not running

## What NOT to Do

- ❌ Add unnecessary dependencies
- ❌ Require complex setup
- ❌ Depend on external services
- ❌ Over-engineer or premature abstraction (YAGNI)
- ❌ Sacrifice performance targets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matsumo-and)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/matsumo-and)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
