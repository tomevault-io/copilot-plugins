## ctx

> `ctx` is an extremely fast local semantic codebase search engine written in Rust. It provides semantic and full-text code search via a CLI, designed for both human developers and AI coding agents.

# AGENTS.md — Project conventions for ctx

## Project Overview

`ctx` is an extremely fast local semantic codebase search engine written in Rust. It provides semantic and full-text code search via a CLI, designed for both human developers and AI coding agents.

## Tech Stack

- **Language**: Rust (edition 2021)
- **Embeddings**: Candle (HuggingFace pure-Rust ML) + MiniLM-L6-v2
- **Full-text**: Tantivy 0.22
- **Vector search**: `instant-distance` (HNSW, mmap-backed)
- **Code chunking**: `tree-sitter` with compile-time grammars
- **File walking**: `ignore` crate (ripgrep-compatible)
- **CLI**: `clap` with derive macros
- **Serialization**: `serde`, `serde_json`, `bincode`
- **Async**: `tokio` (for watch mode and future HTTP server)
- **File watching**: `notify`

## Code Structure

```
src/
├── main.rs           # CLI entry point (clap, subcommand dispatch)
├── lib.rs            # Library root (re-exports all modules)
├── config.rs         # Configuration (.ctx/config.ctx, TOML format)
├── download.rs       # Model auto-download from HuggingFace Hub
├── cli/
│   ├── mod.rs        # CLI module re-exports
│   ├── index.rs      # `ctx index` command
│   ├── search.rs     # `ctx search` command
│   ├── watch.rs      # `ctx watch` command
│   ├── serve.rs      # `ctx serve` command
│   ├── status.rs     # `ctx status` command
│   ├── config.rs     # `ctx config` command
│   └── clean.rs      # `ctx clean` command
├── indexer/
│   ├── mod.rs        # Indexer re-exports
│   ├── walker.rs     # File traversal (.gitignore-aware)
│   ├── chunker.rs    # tree-sitter AST-aware code chunking
│   ├── embedder.rs   # Candle BERT embedding generation
│   ├── fast_embed.rs # NGramEmbedder character n-gram fallback
│   ├── incremental.rs # SHA-256 hash-based change detection
│   └── builder.rs    # Full indexing pipeline orchestration
├── search/
│   ├── mod.rs        # Search re-exports
│   ├── vector.rs     # HNSW vector similarity search
│   ├── fulltext.rs   # Tantivy full-text search
│   └── hybrid.rs     # Reciprocal rank fusion (HybridResult)
├── server/
│   ├── mod.rs        # Server module re-exports
│   └── mcp.rs        # MCP JSON-RPC server (Claude Desktop/Cursor)
└── store/
    ├── mod.rs        # IndexPaths, directory layout
    └── metadata.rs   # Manifest, FileEntry
```

## Conventions

### Documentation
- Every module has a `//!` doc comment explaining its purpose and architecture
- Every public function has a `///` doc comment with arguments and behavior
- Use `NOTE:` for future work items
- Use `Future:` for planned features with rationale
- Use `TODO:` for known issues that need fixing

### Error Handling
- Use `anyhow::Result` for application-level error propagation
- Use `.with_context()` for descriptive error messages on fallible operations
- Use `thiserror` for custom error types in library code

### Testing
- Unit tests live in `#[cfg(test)] mod tests` at the bottom of each file
- Test data should be in-memory or use temp directories (`std::env::temp_dir()`)
- Clean up temp data in tests

### Performance
- Use `rayon` for CPU-bound parallelism (chunking, embedding)
- Use `memmap2` for zero-copy file access
- Use `Cow` and `Arc` where appropriate for shared data
- Avoid unnecessary clones in hot paths

### Safety
- Unsafe code is only used in `memmap2` calls with documented invariants
- No unwrap() in production code — use `.context()` or `?`
- File operations use atomic writes (write to tmp, rename)

## Build Commands

```bash
cargo build              # Debug build
cargo build --release    # Release build (with LTO)
cargo test               # Run all tests (89: 79 unit + 10 integration)
cargo test -- --nocapture  # Run tests with output
cargo clippy             # Lint
RUST_LOG=debug cargo run -- index  # Debug mode with logging
```

## Index Storage Format

Index lives at `.ctx/index/` in the project root:

```
.ctx/
├── config.ctx          # Project configuration (TOML, optional)
├── index/
│   ├── manifest.json   # Metadata (version, chunk count, timestamps, files)
│   ├── chunks.bin      # Chunk data (bincode serialized)
│   ├── embeddings.bin  # Embedding vectors (raw f32, memory-mapped)
│   ├── hnsw.bin        # HNSW graph (instant-distance format)
│   └── fulltext/       # Tantivy index directory
└── tmp/                # Temporary files during indexing
```

## Future Work (Flagged in Code)

Search code for `NOTE:` and `Future:` comments for planned improvements:
- ~~HuggingFace model auto-download~~ ✓ Done (v0.1)
- ~~Swap `ort` for `candle` (pure Rust) to remove C dependency~~ ✓ Done (v0.2)
- ~~Incremental indexing~~ ✓ Done (v0.2)
- ~~MCP server for Claude Desktop / Cursor~~ ✓ Done (v0.4)
- ~~Pi extension / skill integration~~ ✓ Done (v0.4)
- ~~CI/CD (GitHub Actions)~~ ✓ Done (v0.4)
- ~~Benchmark suite~~ ✓ Done (v0.4)
- Daemon async I/O + streaming responses
- Streaming writes for large codebases (>10M lines)
- Quantized embeddings (int8 product quantization)
- `ctx serve` HTTP API
- Dynamic tree-sitter grammar loading

---
> Source: [nwalsh2102/ctx](https://github.com/nwalsh2102/ctx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
