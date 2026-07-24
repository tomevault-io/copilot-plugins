## tlparse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building
- `cargo build` - Build the Rust binary
- `cargo build --release` - Build optimized release version

### Testing
- `cargo test` - Run all tests
- `cargo test --verbose` - Run tests with detailed output
- `cargo test tests/integration_test.rs` - Run specific test file

### Development
- `cargo run -- <input_dir> -o <output_dir>` - Run tlparse with arguments
- `cargo run -- --help` - Show command help
- `cargo check` - Fast compile check without generating binary

### Python Package (via maturin)
- `pip install maturin` - Install build system for Python bindings
- `maturin develop` - Install development version in current Python environment
- `maturin build` - Build Python wheel

### Release Process
1. Update version in `Cargo.toml`
2. Run `cargo update` to update `Cargo.lock`
3. Create release commit and tag (triggers PyPI release)
4. Run `cargo publish` to publish to crates.io

## Code Architecture

### Core Components

**Main Library (`src/lib.rs`)**
- `parse_path()` - Primary entry point that processes TORCH_LOG files
- Handles glog parsing, JSON deserialization, and coordinates all parsers
- Returns `ParseOutput` (vector of file paths and contents to write)
- Supports both regular analysis mode and export mode

**Type System (`src/types.rs`)**
- `Envelope` - Main structured log entry container with all possible metadata fields
- `CompileId` - Unique identifier for compilation attempts (frame_id/frame_compile_id/attempt)
- `CompilationMetricsMetadata` - Performance and failure tracking data
- `StackSummary` and `FrameSummary` - Stack trace representation
- Various metadata types for different log entry types

**Parser Framework (`src/parsers.rs`)**
- `StructuredLogParser` trait - Implement to create custom analysis parsers
- `get_metadata()` - Filter which log entries a parser processes
- `parse()` - Transform log data into output files or links
- `ParserOutput` enum - File, GlobalFile, or Link outputs

**CLI Interface (`src/cli.rs`)**
- Built with `clap` derive macros
- Primary command: `tlparse <input_log_file> -o <output_directory>`
- Supports custom parsers, export mode, and various analysis options

### Key Data Flow

1. **Log Parsing**: Reads TORCH_LOG files line by line using glog regex
2. **Deserialization**: Converts JSON payloads to `Envelope` structs
3. **Parser Execution**: Runs all registered parsers on matching envelopes
4. **Output Generation**: Collects files and generates HTML index with navigation
5. **File Writing**: Saves all generated content to output directory

### Parser System

The extensible parser system allows custom analyses:
- Default parsers handle common log types (graphs, metrics, guards)
- Each parser filters relevant envelopes via `get_metadata()`
- Parsers return files (HTML, JSON, code) or external links
- Results are organized by `CompileId` in the output directory

### Template System

Uses `TinyTemplate` for HTML generation:
- Templates defined in `src/templates.rs`
- CSS and JavaScript embedded as static strings
- Supports both regular and export modes with different templates
- Index pages provide navigation between compilation attempts

### Development Notes

- Uses `fxhash` for performance-critical hash maps
- Intern table for string deduplication in stack traces  
- Progress bars via `indicatif` for large log processing
- `anyhow` for error handling throughout
- Supports both Rust binary and Python package builds via `maturin`

---
> Source: [meta-pytorch/tlparse](https://github.com/meta-pytorch/tlparse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
