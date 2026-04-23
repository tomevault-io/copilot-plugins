## multiread

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust library called `multiread` that provides a high-performance JSONL (JSON Lines) parser. The main component is `MultiJsonlByteParser`, which can efficiently parse JSONL files by iterating through lines while correctly handling JSON strings that contain newline characters.

## Architecture

The codebase consists of:

- **`src/lib.rs`**: Contains the main `MultiJsonlByteParser<'a, AsSlice>` struct that implements an iterator over JSONL byte slices. It handles quote counting to correctly parse JSON strings containing embedded newlines.

- **`src/utils.rs`**: Contains SIMD (AVX2) optimized utilities for pattern matching, specifically `find_all_matches_m256()` which uses 256-bit vector instructions to find byte patterns efficiently. This is experimental work for performance optimization.

- **`tests/`**: Contains integration tests and test data including a sample JSONL file with poker game data that includes embedded newlines within JSON string values.

## Development Commands

### Basic Rust Development
```bash
# Build the project
cargo build

# Run tests
cargo test

# Run tests with verbose output
cargo test -- --nocapture

# Check code without building
cargo check

# Format code
cargo fmt

# Run clippy linter
cargo clippy
```

### Nix Development Environment
The project uses Nix flakes for development environment management:

```bash
# Enter development shell (if using Nix)
nix develop

# The flake provides additional tools:
# - cargo-expand: View macro expansions
# - cargo-nextest: Fast test runner
# - cargo-insta: Snapshot testing
```

### Advanced Testing
```bash
# Run with nextest (if in nix shell)
cargo nextest run

# Run individual test
cargo test test_small_example_jsonl_len

# View expanded macros
cargo expand
```

## Key Implementation Details

- The parser uses unsafe code with `get_unchecked()` for performance
- Quote counting (`quote_counter % 2`) determines if currently inside a JSON string
- Handles escaped quotes by tracking the previous byte (`self.last`)
- SIMD optimizations in `utils.rs` are experimental and use AVX2 intrinsics
- The project uses Rust 2024 edition

## Testing

The test suite uses the `pretty_assertions` crate for better diff output. Test data includes a JSONL file with embedded newlines in JSON string values to verify correct parsing behavior.

When running tests, the parser should correctly handle:
- JSON strings containing newline characters
- Escaped quotes within strings
- Multiple JSONL records

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SimonTheoret) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
