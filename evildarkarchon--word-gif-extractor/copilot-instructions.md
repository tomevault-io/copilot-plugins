## word-gif-extractor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Rust CLI tool that extracts image files from Microsoft Word (.docx) documents by treating them as ZIP archives. Supports multiple image formats: jpg, jpeg, png, gif, bmp, tiff, svg, wmf, emf, webp, ico.

## Build Commands

```bash
# Build release binary
cargo build --release

# Run directly via cargo
cargo run -- "path/to/document.docx"
cargo run -- --input "path/to/document.docx" --output "output/folder"

# Process a directory of .docx files
cargo run -- "path/to/folder"

# Recursive directory processing
cargo run -- "path/to/folder" -r

# Filter by format (comma-separated)
cargo run -- "document.docx" -f png,gif

# Extract only cover images from EPUB files
cargo run -- "book.epub" -c
cargo run -- "path/to/epubs" -r --cover-only

# Run tests
cargo test
```

The release binary is output to `target/release/word-image-extractor.exe`.

## Architecture

Single-file application (`src/main.rs`) with straightforward flow:
1. Parse CLI arguments with `clap` (supports positional or `--input` flag)
2. Determine target image extensions (all supported or user-filtered via `-f`)
3. If input is a file: process it directly; if directory: iterate over .docx files (optionally recursive with `-r`)
4. Open each .docx as a ZIP archive and scan for matching image extensions
5. Extract images to output directory, renamed as `{docname}_{n}.{ext}` (or `{docname}.{ext}` if only one)

## Dependencies

- **zip**: Archive traversal for .docx files
- **clap**: CLI argument parsing with derive macros
- **anyhow**: Error handling with context
- **walkdir**: Recursive directory traversal

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evildarkarchon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
