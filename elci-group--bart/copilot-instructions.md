## bart

> Bart is a Rust CLI tool for visualizing directory structures with file sizes and file counts. It acts like a combination of `tree` and `du`, providing a colorful, hierarchical view of disk usage.

# Bart - Project Guide

## Overview
Bart is a Rust CLI tool for visualizing directory structures with file sizes and file counts. It acts like a combination of `tree` and `du`, providing a colorful, hierarchical view of disk usage.

## Codebase Structure
- **Language**: Rust (2021 edition)
- **Entry Point**: `src/main.rs`
- **Config**: `Cargo.toml`

### Key Components
- **`Args` struct**: Defines CLI arguments using `clap`.
- **`Node` struct**: Represents a file or directory in the tree, containing size, file count, and children.
- **`scan` function**: Recursively traverses the directory structure to build the `Node` tree.
- **`print_recursive` function**: Handles the visualization, including ASCII tree generation and size bars.

## Development

### Essential Commands
- **Build**: `cargo build`
- **Run**: `cargo run -- [args]`
  - Example: `cargo run -- --depth 2`
  - Example: `cargo run -- -n 5` (show top 5 items)
- **Check**: `cargo check`
- **Test**: `cargo test`

### Code Style & Conventions
- **Formatting**: Standard Rust (`cargo fmt`).
- **Error Handling**: 
  - File permission errors during scan are currently silenced/ignored to allow the scan to continue (see `scan` function).
  - Main returns `()` and handles top-level errors with `eprintln!` and `exit(1)`.
- **Imports**: Grouped std imports, followed by external crates.

## Dependencies
- **clap**: Command line argument parsing.
- **colored**: Terminal output coloring.
- **humansize**: Human-readable file size formatting.
- **term_size**: Terminal dimension detection.
- **unicode-width**: Correct string width calculation for alignment.
- **walkdir**: Listed in `Cargo.toml` but `fs::read_dir` is currently used for traversal.

## Gotchas & Patterns
- **Manual Recursion**: The directory scanning is implemented manually with `fs::read_dir` rather than using `walkdir` iterator, likely to have fine-grained control over the tree structure construction.
- **Visual Alignment**: The tree visualization calculates padding manually using `unicode-width` to ensure bars align correctly even with unicode characters.
- **Depth Coloring**: Colors cycle based on depth (`depth % 6`).
- **Unused Dependency**: `walkdir` appears to be in `Cargo.toml` but not currently used in `src/main.rs`.

---
> Source: [elci-group/bart](https://github.com/elci-group/bart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
