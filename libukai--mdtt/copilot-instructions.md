## mdtt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MDict Utils is a Python library for packing and unpacking MDict dictionary files (.mdx/.mdd). It supports reading/writing MDict Version 2.0 and reading MDict Version 3.0. The project is built using modern Python 3.13+ with uv as the package manager and build backend.

**Major Architecture:** The project uses a modern subcommand architecture with TOML-based metadata management, replacing the old argument-based interface.

## Development Commands

### Package Management
- `uv sync` - Install dependencies and create virtual environment
- `uv add <package>` - Add new dependency
- `uv remove <package>` - Remove dependency

### Code Quality
- `uv run ruff check` - Run linting (pycodestyle, pyflakes, isort, security checks, etc.)
- `uv run ruff format` - Format code
- `uv run pyright` - Run type checking

### Testing
- `tests/run_tests.sh all` - Run all tests using the test runner
- `tests/run_tests.sh unit` - Run unit tests only
- `tests/run_tests.sh integration` - Run integration tests with real files
- `tests/run_tests.sh -c` - Run tests with coverage report
- `uv run pytest` - Direct pytest execution
- `uv run pytest -m "not slow"` - Skip slow tests
- `tests/test_integration.sh` - Run shell-based integration tests

### Running the Tool (New Subcommand Architecture)
- `uv run mdict --help` - Show all available commands
- `uv run mdict extract dict.mdx` - Extract dictionary
- `uv run mdict pack -a source.txt dict.mdx` - Pack dictionary
- `uv run mdict query word dict.mdx` - Query word
- `uv run mdict info dict.mdx` - Show dictionary information
- `uv run mdict keys dict.mdx` - List dictionary keys
- `uv run mdict convert txt-to-db dict.txt dict.db` - Convert formats

## Architecture

### Core Components

1. **Reader Module** (`src/mdict_utils/reader.py`)
   - Handles unpacking .mdx/.mdd files to text/database formats
   - Supports querying individual entries and extracting metadata
   - Uses `base.readmdict.MDX` and `base.readmdict.MDD` classes for low-level file parsing

2. **Writer Module** (`src/mdict_utils/writer.py`)
   - Handles packing text files or databases into .mdx/.mdd format
   - Supports multiple input sources (txt files, SQLite databases, directories)
   - Uses `base.writemdict.MDictWriter` for low-level file writing

3. **Metadata Module** (`src/mdict_utils/metadata.py`) - **NEW**
   - TOML-based metadata management system
   - Automatic metadata file detection and validation
   - Conversion between TOML format and MDX headers
   - MetadataManager class for intelligent caching and operations

4. **Subcommand Architecture** (`src/mdict_utils/commands/`) - **NEW**
   - Modern CLI with individual command modules
   - `extract.py`, `pack.py`, `query.py`, `info.py`, `keys.py`, `convert.py`
   - Each command inherits from `BaseCommand` with consistent interface
   - Rich help system and error handling

5. **Main Entry Point** (`src/mdict_utils/__main__.py`) - **REDESIGNED**
   - Modern argparse with subcommands (like git, docker, etc.)
   - Global options (--verbose, --quiet) and per-command options
   - Comprehensive help and usage examples
   - Graceful error handling with user-friendly messages

6. **Base Classes** (`src/mdict_utils/base/`)
   - Low-level MDict file format implementation
   - Handles compression, block structures, and binary format details

### File Format Support

- **MDX files**: Dictionary entries (word definitions)
- **MDD files**: Media resources (images, audio, etc.)
- **Database format**: SQLite3 with `mdx` and `mdd` tables for faster access
- **Text format**: UTF-8 encoded key-value pairs

### Key Features

- **TOML Metadata Management**: User-friendly `.meta.toml` files for dictionary metadata
- **Subcommand Architecture**: Modern CLI with `extract`, `pack`, `query`, `info`, `keys`, `convert` commands
- **Automatic Metadata Detection**: Finds `.meta.toml` files automatically based on source file names
- **Multiple Output Formats**: Text, database, split files, JSON, TOML
- **Rich Information Display**: Beautiful formatted output with color and structure
- **Progress Tracking**: Visual progress bars for long operations
- **Support for MDict Versions**: 1.2, 2.0, 3.0 with automatic version detection
- **Advanced Features**: Encrypted dictionaries, substyle support, compact HTML conversion
- **Comprehensive Testing**: Unit, integration, and shell-based tests with real dictionary files

## Testing Structure

Tests are organized in the `tests/` directory:
- `test_packing.py` - Legacy packing tests
- `test_unpacking.py` - Legacy unpacking tests  
- `test_subcommands.py` - **NEW** Tests for subcommand architecture
- `test_metadata.py` - **NEW** Tests for TOML metadata functionality
- `conftest.py` - **NEW** Pytest configuration and shared fixtures
- `test_config.toml` - **NEW** Configuration for real dictionary test files
- `run_tests.sh` - **NEW** Smart test runner with multiple execution modes
- `test_integration.sh` - **NEW** Shell-based integration testing
- `README.md` - **NEW** Comprehensive testing documentation

### Test Categories and Markers:
- `@pytest.mark.unit` - Fast tests with synthetic data
- `@pytest.mark.integration` - Tests with real dictionary files
- `@pytest.mark.subcommands` - Subcommand architecture tests
- `@pytest.mark.metadata` - TOML metadata functionality tests
- `@pytest.mark.slow` - Performance/database conversion tests
- `@pytest.mark.requires_real_files` - Tests requiring configured dictionary files

## Dependencies

- **tqdm**: Progress bar display
- **xxhash**: Fast hashing for MDict format
- **tomli-w**: TOML writing support (Python 3.11+ has built-in tomllib for reading)
- **pytest ecosystem**: Testing framework with coverage and mocking support
- **ruff**: Modern Python linter and formatter
- **pyright**: Static type checker

## TOML Metadata Format

The new metadata system uses user-friendly TOML files:

```toml
[dictionary]
title = "My Dictionary"
description = """
A comprehensive dictionary with detailed definitions.
Supports multiple languages and advanced features.
"""
author = "Dictionary Author"
email = "author@example.com"
website = "https://example.com"
copyright = "© 2024 Author"
version = "2.0"  # MDX format version (1.2, 2.0, 3.0)

[advanced]  # Optional technical parameters
encoding = "UTF-8"    # Default: UTF-8
key_size = 32        # KB, default: 32
record_size = 64     # KB, default: 64
```

### Metadata File Discovery:
- `source.txt` → looks for `source.meta.toml`
- `mydict` → looks for `mydict.meta.toml`
- Manual specification: `--meta custom.toml`

## Migration from Legacy Interface

### Old vs New Command Syntax:

```bash
# Old argument-based interface (deprecated)
mdict -a source.txt -t title.txt -d desc.txt output.mdx
mdict -x dict.mdx -o ./output
mdict -q word dict.mdx
mdict -m dict.mdx

# New subcommand interface
mdict pack -a source.txt output.mdx              # Auto-detects metadata
mdict pack -a source.txt output.mdx -m meta.toml # Explicit metadata
mdict extract dict.mdx -o ./output -e            # Extract with metadata export
mdict query word dict.mdx
mdict info dict.mdx                              # Rich formatted output
mdict info dict.mdx --format json               # JSON output
```

### Benefits of New Architecture:
- **Cleaner syntax**: Clear subcommands instead of cryptic flags
- **Better help**: `mdict extract --help` for command-specific help
- **Rich output**: Formatted info display with colors and structure
- **Extensibility**: Easy to add new commands without parameter conflicts
- **Consistency**: Similar to modern tools like git, docker, etc.

---
> Source: [libukai/mdtt](https://github.com/libukai/mdtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
