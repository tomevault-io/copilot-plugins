## selfspy3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Selfspy is a comprehensive activity monitoring suite with implementations in multiple programming languages. It continuously monitors:
- Keystrokes (encrypted for security)
- Mouse movements and clicks
- Active window titles and processes
- Activity periods

This is a multi-language project with implementations in Python (primary), Rust, Elixir/Phoenix, Objective-C, Nim, Crystal, Zig, and more. The Python implementation is the most mature and feature-complete.

## Project Structure

```
selfspy3/
├── python/           # 🐍 Main Python implementation (most mature)
├── rust/             # 🦀 Rust implementation with GUI
├── elixir/           # 🔥 Phoenix web interface
├── objective-c/      # 🍎 macOS native widgets
├── nim/              # ⚡ Nim compiled implementation
├── crystal/          # 💎 Crystal implementation
├── zig/              # ⚡ Zig high-performance implementation
├── [other langs]/    # Additional language implementations
├── shared/           # Shared resources and scripts
├── tools/            # Development tools
└── dev               # Multi-language development helper script
```

## Multi-Language Development

Use the `./dev` helper script to work with different implementations:

```bash
# Work with Python (most mature implementation)
./dev python uv run selfspy start
./dev python uv run pytest

# Work with Rust implementation
./dev rust cargo build --release
./dev rust cargo run --bin selfspy-gui

# Work with Elixir/Phoenix web interface
./dev elixir mix phx.server
./dev elixir mix test

# Work with other implementations
./dev objective-c make all          # macOS widgets
./dev nim nimble build              # Nim compiled
./dev crystal shards build         # Crystal compiled
./dev zig zig build                 # Zig performance

# Language-agnostic commands
./dev test                          # Run all tests
./dev build                         # Build all implementations
```

**Note**: You are currently on the `elixir-phoenix-implementation` branch, which indicates active development on the Elixir/Phoenix web interface implementation.

## Python Implementation Commands

The following commands are specific to the Python implementation (located in `python/` directory):

### Setup and Installation

**Option 1: Using uv (recommended for development)**
```bash
# Install dependencies
uv sync --group dev

# Install with macOS extras for full functionality
uv sync --group dev --extra macos

# Alternative uv installation
uv pip install .
```

**Option 2: Using pip (standard Python)**
```bash
# Quick automatic installation
python3 install.py

# Manual installation - basic
pip3 install -r requirements.txt
pip3 install -e .

# Manual installation - macOS with full functionality
pip3 install -r requirements-macos.txt
pip3 install -e .

# Development setup
pip3 install -r requirements-dev.txt
pip3 install -e .
```

### Running the Application

**With uv (from development environment):**
```bash
# Start monitoring (will guide through permission setup on macOS)
uv run selfspy start

# Start monitoring with custom data directory
uv run selfspy start --data-dir /path/to/data

# Start monitoring without text logging
uv run selfspy start --no-text

# View enhanced statistics with rich visualizations 🎨
uv run selfviz enhanced

# View activity timeline
uv run selfviz timeline --days 1

# Terminal command analytics 🔧
uv run selfterminal commands --days 7
uv run selfspy terminal sessions --days 30

# Check macOS permissions manually
uv run selfspy check-permissions
```

**Direct commands (after installation):**
```bash
# Start monitoring
selfspy start

# View statistics
selfstats
selfviz enhanced

# Terminal analytics
selfterminal commands --days 7
selfspy terminal projects --days 30

# Check permissions (macOS)
selfspy check-permissions
```

### Development and Testing

**With uv (recommended for development):**
```bash
# Run tests
uv run pytest

# Run with coverage (default from pytest.ini)
uv run pytest --cov=src

# Run a single test file
uv run pytest tests/test_models.py

# Linting and formatting
uv run black src/ tests/
uv run isort src/ tests/
uv run ruff check src/ tests/
uv run mypy src/

# Pre-commit hooks
uv run pre-commit run --all-files
```

**Direct commands (after pip installation):**
```bash
# Run tests
pytest

# Linting and formatting
black src/ tests/
isort src/ tests/
ruff check src/ tests/
mypy src/

# Pre-commit hooks
pre-commit run --all-files
```

## Architecture Overview

### Core Components

**Entry Points (`python/src/cli.py` and `python/src/stats.py`)**
- Uses Typer for modern CLI interface with Rich output
- Main commands: `start` (monitoring), `stats` (analysis), `check-permissions`
- Handles async event loops and signal handling

**Activity Monitor (`python/src/activity_monitor.py`)**
- Central orchestrator using async patterns
- Manages platform-specific trackers (keyboard, mouse, window)
- Includes terminal command tracking (background task)
- Maintains live dashboard during monitoring
- Buffers keystrokes before storage to reduce I/O

**Terminal Tracking (`python/src/terminal_tracker.py` and `python/src/terminal_stats.py`)**
- Monitors shell command history files (bash, zsh, fish)
- Tracks commands by working directory and project context
- Categorizes commands by type (git, npm, python, etc.)
- Detects project types and git branches automatically
- Provides rich analytics and visualizations for development workflows

**Data Layer**
- `python/src/models.py`: SQLAlchemy 2.0 models with modern mapped_column syntax
- `python/src/activity_store.py`: Database operations with async session management
- `python/src/encryption.py`: Handles keystroke encryption using cryptography library

**Platform Abstraction (`python/src/platform/`)**
- `base.py`: Abstract interfaces for cross-platform support
- `darwin.py`: macOS-specific implementations using PyObjC
- `fallback.py`: Basic implementations for other platforms
- Platform detection and graceful degradation

### Key Design Patterns

- **Async/Await**: All I/O operations use asyncio for better performance
- **Dependency Injection**: Settings and stores passed to components
- **Platform Abstraction**: Interface-based design allows platform-specific implementations
- **Encryption by Default**: All keystroke data encrypted unless explicitly disabled
- **Modern Python**: Uses type hints, dataclasses, and modern SQLAlchemy patterns

### Database Schema

Uses SQLAlchemy 2.0 with declarative base and relationships:
- `Process`: Application information with macOS bundle IDs
- `Window`: Window metadata including geometry and screen info
- `Keys`: Encrypted keystroke data with counts
- `Click`: Mouse events with coordinates and movement tracking
- `TerminalSession`: Terminal session metadata (shell type, working directory)
- `TerminalCommand`: Individual commands with context (git branch, project type, exit codes)

All tables include timestamp mixins and proper indexing for performance.

### Configuration (`python/src/config.py`)

Uses Pydantic Settings for type-safe configuration:
- Environment variable support with `SELFSPY_` prefix
- Path validation and automatic directory creation
- Platform-specific settings for macOS features
- Privacy and security controls

## macOS Integration

The project has extensive macOS support requiring:
- Accessibility permissions for window tracking
- Optional Screen Recording permissions for screenshots
- PyObjC frameworks for native API access
- Bundle ID tracking for better app identification

Use `uv sync --group dev --extra macos` for full macOS functionality.

## Security Considerations

- All keystroke data encrypted by default using industry-standard cryptography
- Configurable exclusions for sensitive applications
- Privacy mode available for restricted logging
- Local storage only - no network transmission
- Secure password handling via keyring integration

## Testing and Code Quality

- Uses pytest with async support and coverage reporting
- Code formatting with Black and import sorting with isort
- Type checking with mypy
- Linting with ruff
- Pre-commit hooks for code quality enforcement

The codebase follows modern Python conventions with comprehensive type hints and documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nuin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nuin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
