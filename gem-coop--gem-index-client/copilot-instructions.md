## gem-index-client

> A Rust async HTTP client library for syncing the RubyGems compact index format. Implements a caching strategy using ETags, range requests, and SHA256 verification for bandwidth-efficient updates.

# CLAUDE.md

## Project Overview

A Rust async HTTP client library for syncing the RubyGems compact index format. Implements a caching strategy using ETags, range requests, and SHA256 verification for bandwidth-efficient updates.

## Commands

### Development

```bash
cargo test
cargo build
cargo check
cargo clippy --fix --all-targets --all-features
cargo fmt
```

### Examples

```bash
# Run any example
cargo run --example full_workflow
```

## Architecture

### Core Design Pattern

The library uses a caching strategy coordinated by `IndexClient`.
Each layer is tolerant of missing information from other layers, allowing graceful degradation.

#### 1. **Versions File**

Downloads `versions` file using ETag/range requests; extracts info file checksums

#### 2. Cache Verification

Checks if cached info files match expected checksums (skip download if valid)

#### 3. Info Files

Fetches missing/stale info files using range requests with HTTP digest verification

### Module Structure

```
lib.rs                   # Public API and error types
├── index_client.rs      # High-level orchestration (3-layer caching)
├── updater.rs           # Mid-level ETag/range request logic
├── http_fetcher.rs      # Low-level HTTP with retry/backoff
├── storage/             # Storage abstraction
│   ├── mod.rs           # Blob, BlobMetadata, Storage trait
│   ├── filesystem.rs    # Filesystem storage with metadata JSON
│   └── memory.rs        # In-memory storage for testing
├── names.rs             # Gem names list
├── versions.rs          # Versions file, extract checksums
└── info.rs              # Gem info metadata
```

### Key Abstractions

**IndexClient**: Builder-pattern client coordinating the caching strategy.

**Updater**: Handles incremental updates using ETags and range requests. Two methods:

- `update()`: Conditional request with ETag (304 Not Modified or 206 Partial Content)
- `fetch()`: Unconditional full download

**Storage trait**: Abstraction over storage backends (filesystem, S3, memory). Works with `Blob` objects containing content + metadata (etag, md5, sha256).

### Performance Characteristics

- **I/O-bound workload**: Fetching remote files and reading files from disk
- **CPU-bound workload**: Calculating checksums of files

### Compact Index Format

The RubyGems compact index has three file types.

1. **`names`**: Newline-separated list of gem names
2. **`versions`**: All versions for each gem with checksums
   - Format: `gem_name version1,version2 checksum`
   - Size: Often 20MB or larger
3. **`info/{gem}`**: Detailed version info (dependencies, ruby version, platform)
   - Format: One version per line with dependencies and constraints

All files may have a YAML-like header ending with `---\n`. Parsers skip this header.

## Testing Notes

- Use `tempfile::TempDir` for filesystem tests to ensure cleanup
- Mock the `Fetcher` trait for unit testing HTTP interactions without network calls

## Code Patterns

### Async Patterns

All I/O operations are async using `tokio`.
Use `futures::stream::iter().buffer_unordered(N)` for parallel processing with concurrency limits.

The `#[tokio::main]` macro provides the default runtime (thread count = CPU cores), which is optimal for I/O-bound workloads.

### Storage Implementations

When implementing `Storage`:

- `read_metadata()` should be optimized to avoid loading full content
- `write_blob()` must persist all metadata (etag, md5, sha256) for verification
- Use `async_trait` for async methods in traits
- `FilesystemStorage` uses JSON sidecar files (`.meta.json`) for metadata

## Common Gotchas

- **Range request overlap**: Partial content (206) includes 1 byte overlap with existing content, which is skipped: `blob.append(&response.body[1..])` to avoid empty responses which have historically caused errors.
- **ETag wrapping**: Strip `W/` prefix and quotes from ETags before storing
- **Case-insensitive headers**: Use `Response::get_header()` for case-insensitive lookup
- **Empty response handling**: 304 Not Modified returns empty body - return cached blob without re-verification

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gem-coop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
