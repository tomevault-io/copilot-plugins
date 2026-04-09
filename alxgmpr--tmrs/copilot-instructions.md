## tmrs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`tmrs` is a Rust CLI tool that runs a command multiple times and reports average execution time with standard deviation — like an enhanced `time` command. Published on crates.io.

## Build & Development Commands

```bash
cargo build --release       # Build optimized binary
cargo test --verbose        # Run tests
cargo clippy --all-targets --all-features  # Lint
cargo fmt --all             # Format code
```

CI (GitHub Actions on push to master / PRs) runs fmt check, clippy, and build+test across ubuntu/macos/windows.

## Architecture

Single-file application in `src/main.rs`. All logic lives there:

- **CLI parsing**: `clap` with derive macros (`Args` struct)
- **Core loop**: Runs the target command N times (default 5, `-n` flag), collecting `Duration` samples
- **Statistics**: `std_deviation()` computes sample standard deviation (N-1); `mean()` from the `statistics` crate
- **Output**: Formatted average ± stddev

Key dependencies: `clap` (CLI), `statistics` (math), `log`/`env_logger` (debug logging).

## Contribution Guidelines

- All code must pass `cargo clippy` and `cargo fmt` before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alxgmpr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alxgmpr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
