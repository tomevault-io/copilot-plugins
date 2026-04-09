## learning-rust

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a personal Rust learning repository started on August 17, 2022. It contains exercises and examples from various Rust books and courses, organized into topic-specific modules. Each module demonstrates different Rust concepts and features.

## System Requirements

When working with this codebase on Debian-based systems, the following packages must be installed:
- libssl-dev
- pkg-config
- fd-find
- zlib1g-dev

## Project Structure

This is a Cargo workspace with both a library (`common`) and binary (`learning_rust`):

- **Library crate**: `src/lib.rs` exports all learning modules as `pub mod`
- **Binary crate**: `src/main.rs` calls `index(false)` on each module to run examples
- **Module pattern**: Each topic lives in `src/[topic]/` with a `mod.rs` that exports an `index(bool)` function
  - Pass `true` to `index()` to enable output for that module
  - Pass `false` to skip execution (default for all modules in main.rs)

## Common Development Commands

### Build the project
```bash
cargo build
```

### Run the main binary (executes all module index functions)
```bash
cargo run
```

### Run with a specific module enabled
Edit `src/main.rs` and change the relevant module's `index(false)` to `index(true)`, then:
```bash
cargo run
```

### Check code without building
```bash
cargo check
```

### Run tests
```bash
cargo test
```

### Build and run in release mode
```bash
cargo run --release
```

## Working with Modules

When adding new learning examples:

1. Create a new directory under `src/` for the topic (e.g., `src/new_topic/`)
2. Add a `mod.rs` file with an `index(show: bool)` function
3. Export the module in `src/lib.rs`: `pub mod new_topic;`
4. Call it from `src/main.rs`: `common::new_topic::index(false);`

The `index(show: bool)` pattern allows selective execution of examples without commenting out code.

## Dependencies

Key crates used across examples:
- `rand`: Random number generation
- `futures` & `async-std`: Async/await examples
- `strum` & `strum_macros`: Enum utilities
- `chrono` & `humantime`: Time handling
- `thiserror`: Error handling
- `num`: Numeric types including complex numbers
- `polars`: DataFrame operations for ML examples
- `libc`: FFI and system call examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rustnation)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rustnation)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
