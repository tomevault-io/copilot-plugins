## arqfastrestore

> Guidance for AI coding agents operating in this repository.

# AGENTS.md — arqfastrestore

Guidance for AI coding agents operating in this repository.

---

## Project Overview

**arqfastrestore** is a fast, parallel restore tool for [Arq](https://www.arqbackup.com/) backups, written in Rust.

- Supports **Arq5** and **Arq7** backup formats
- Decrypts AES-CBC encrypted backups (HMAC-SHA1/SHA256, PBKDF2 key derivation)
- Decompresses LZ4, zlib/gzip, and zstd blobs
- Restores files in parallel using Rayon
- Preserves timestamps (mtime/atime) and Unix permissions
- Shows a live progress bar with ETA during restore

**Entry point:** `src/main.rs` — `list` and `restore` CLI subcommands.

**Architecture:**
```
src/
  main.rs                    — CLI (list + restore)
  crypto/                    — Arq5/Arq7 decryption
  format/                    — Binary format parsers (Arq5/Arq7 blobs, trees, nodes)
  storage/                   — Storage trait + LocalStorage + BlobReader
  restore/                   — RestoreEngine (parallel restore) + tree_walker + tree_cache
```

See `README.md` for full usage docs and `codemap.md` (if present) for a detailed code map.
Machine-specific deployment details (server access, binary paths, backup locations, passwords) are in `.secrets` (local only, never committed).

**Keeping docs up to date:** When you add features, change CLI flags, or modify the architecture, update both `README.md` and the Architecture section above to reflect the changes.

---

## Build, Lint & Test Commands

```bash
# Build (debug)
cargo build

# Build (release — use this for performance testing)
cargo build --release

# Run all tests
cargo test

# Run a single test by name substring
cargo test select_most_recent

# Run all tests in a specific module
cargo test crypto::arq7

# Run tests with output (don't suppress stdout)
cargo test -- --nocapture

# Lint (treat warnings as errors)
cargo clippy -- -D warnings

# Format
cargo fmt

# Format check only (CI)
cargo fmt -- --check
```

---

## Code Style Guidelines

This is a **pure Rust** codebase — all guidelines below are Rust-specific.

### General Principles
- **Correctness over cleverness** — prefer readable, explicit code.
- **Fail fast** — validate inputs early; return errors close to the source.
- **No silent failures** — every error must be handled or explicitly propagated.
- **Minimal dependencies** — prefer stdlib; add third-party crates only when clearly justified.
- **Graceful degradation in restore** — missing blobs warn instead of aborting the entire restore.

### Naming Conventions
- **Files/modules**: `snake_case` (e.g., `blob_reader.rs`, `tree_walker.rs`)
- **Functions/methods**: `snake_case` (e.g., `parse_backup_record_bytes`)
- **Types/structs/enums**: `PascalCase` (e.g., `BlobLoc`, `RestoreEngine`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `ARQO_MAGIC`, `PBKDF2_ROUNDS`)
- **Variables**: descriptive names; single-letter vars only in short iterators (`i`, `k`, `v`)
- **Test functions**: `condition_expected` style (e.g., `bad_header_rejected`, `select_missing_timestamp_returns_none`)

### Imports
- Group: stdlib → third-party → internal (`crate::`) with a blank line between groups.
- No unused imports — rustc and clippy treat them as errors.
- Prefer explicit imports over glob imports (no `use foo::*`).
- Use `anyhow::{bail, Context, Result}` — import all three together.

### Error Handling
- Use `anyhow::Result<T>` throughout — no custom error types needed.
- Propagate with `?` — never `unwrap()` or `expect()` in production code.
- Add context with `.context("description")?` or `.with_context(|| format!("..."))?`.
- Use `bail!("message")` for early validation failures.
- Use `.map_err(|_| anyhow::anyhow!("message"))?` for opaque errors with a better message.
- `unwrap()` / `expect()` are acceptable only in `#[cfg(test)]` or truly unreachable paths.

### Types and Structs
- Derive only what is needed — `#[derive(Debug, Clone)]` is the default for domain structs.
- Do not add `Copy` unless the type is trivially small and truly semantically copyable.
- Do not add `PartialEq`, `Eq`, `Hash` unless needed for collections or assertions.
- Use `serde` derives only on types that are directly serialized/deserialized; use
  `serde_json::Value` + manual field extraction for ad-hoc parsing.
- Use public fields for data-transfer structs (no getters/setters).
- Model the domain with structs — prefer `BlobLoc` over `(String, u64, u64)`.

### Formatting
- Run `cargo fmt` before every commit — no exceptions.
- Line length: ~100 characters (rustfmt default; occasional overrun is OK for readability).
- No trailing whitespace. Use 4-space indentation (Rust standard; enforced by rustfmt).

### Comments and Documentation
- Module-level docs (`//!` at top of file) — explain the module's purpose and document
  binary format layouts, algorithms, or key concepts.
- Public structs and functions must have `///` doc comments.
- Inline comments explain **why**, not **what** — if the code is self-explanatory, omit.
- Use `TODO(name):` for deferred work, `FIXME:` for known bugs, `HACK:` for workarounds.
- Do not leave commented-out code in commits.

### Testing
- All tests live **inline** in the same source file using `#[cfg(test)] mod tests { ... }`.
- There is no separate `tests/` directory.
- Begin every test module with `use super::*;`.
- Use factory helper functions to build test fixtures — avoid duplicating setup code.
- Mock external dependencies (e.g., `Storage` trait) inside the test module.
- Tests must not depend on the real filesystem or external network; use `tempfile` crate
  or in-memory byte slices.
- Use descriptive test names — `bad_header_rejected` is better than `test_error_case`.
- Aim for at least one error-path test for every public function.

### Parallelism (Rayon)
- Use `rayon::prelude::*` and `par_iter()` / `par_bridge()` for CPU-bound parallel work.
- Prefer collecting errors per-item (warn + count) over aborting on the first failure.
- Share immutable data via `Arc<T>`; avoid `Mutex` unless truly necessary.

### Commit Style
```
<type>(<scope>): <short summary>

Types: feat | fix | refactor | test | docs | chore | perf
Example: feat(restore): add parallel chunk download
```

---

## Repository Map

A full codemap will be available at `codemap.md` in the project root once the project is initialized.

Before working on any task, read `codemap.md` (when it exists) to understand:
- Project architecture and entry points
- Directory responsibilities and design patterns
- Data flow and integration points between modules

For deep work on a specific folder, also read that folder's `codemap.md`.

---
> Source: [stevekstevek/arqfastrestore](https://github.com/stevekstevek/arqfastrestore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
