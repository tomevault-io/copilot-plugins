## cc-app-rust-fileassetbuilder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FileAssetBuilder is a Rust CLI tool that recursively scans a directory and consolidates all files (except excluded types) into a single output file. It uses a blacklist approach - everything is included unless the extension is in the exclusion config.

## Build Commands

```bash
# Build release binary
cargo build --release

# Build debug binary
cargo build

# Run directly with cargo
cargo run -- <input_directory> [-o <output_filename>]
```

## Running the Program

```bash
# Basic usage (outputs to input_directory/fileassets.txt)
./target/release/fileassetbuilder <input_directory>

# Custom output filename (still written to input directory root)
./target/release/fileassetbuilder <input_directory> -o snapshot.txt
```

## Architecture

### Module Structure

- **main.rs** - Entry point, CLI argument parsing (clap), orchestrates other modules
- **config.rs** - Loads/creates config.txt, manages extension blacklist as HashSet
- **scanner.rs** - Recursive directory traversal (walkdir), parallel file reading (rayon)
- **output.rs** - Formats and writes the consolidated output file

### Key Dependencies

- `walkdir` - Recursive directory traversal
- `rayon` - Parallel iterators and custom thread pool for concurrent file processing
- `clap` - CLI argument parsing with derive macros
- `chrono` - Timestamp formatting

### Design Decisions

1. **Recursive scanning** - Traverses all subdirectories
2. **Blacklist approach** - Config lists extensions to EXCLUDE
3. **Dynamic worker pool** - Workers = ceil(file_count / 10)
4. **Parallel reads** - Files are read concurrently with rayon, then sorted for deterministic output
5. **Output location** - Always writes to input directory root, only filename is configurable
6. **Tree-style listing** - Directory list uses `tree /f` style output with box-drawing characters
7. **Absolute paths in Files section** - File content headers show full absolute paths in quotes
8. **Skip if exists** - If output file already exists, skip scanning and exit early

## Configuration

Config file (`config.txt`) is loaded from the same directory as the executable. Auto-created with defaults if missing.

Format:
```
# Comments start with #
.mp4
.png
.exe
```

Extensions must include the leading dot.

## Testing

```bash
# Run on a test directory
./target/release/fileassetbuilder ./test_dir

# Verify output
cat ./test_dir/fileassets.txt
```

---
> Source: [KennethJefferson/cc_app_rust_FileAssetBuilder](https://github.com/KennethJefferson/cc_app_rust_FileAssetBuilder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
