## tvmux

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Build and setup:**
- `make dev` - Set up development environment with venv and pre-commit hooks
- `make install` - Install production dependencies only
- `make clean` - Remove caches and virtual environment

**Testing:**
- `make test` - Run all tests using pytest

**Code quality:**
- Pre-commit hooks automatically run ruff for linting and formatting
- `ruff check .` - Manual linting
- `ruff format .` - Manual code formatting

**Distribution:**
- `make dist` - Build distributable packages
- `make release` - Publish to PyPI

## Architecture Overview

tvmux is a terminal session recorder that creates asciinema cast files from tmux sessions. The project is now at version 0.5.3 with a complete Python rewrite featuring robust client-server architecture.

### Core Components

**CLI Interface (`src/tvmux/cli/`):**
- `main.py` - Main CLI entry point with server and record commands
- `server.py` - Server management commands (start/stop/status)
- `record.py` - Recording control commands

**FastAPI Server (`src/tvmux/server/`):**
- `main.py` - FastAPI application with lifespan management and tmux hook setup
- `state.py` - Global state management for recorders and server configuration
- `routers/` - REST API endpoints:
  - `session.py` - Session management
  - `window.py` - Window management
  - `panes.py` - Pane operations (separate from windows)
  - `recording.py` - Recording control (RESTful with IDs)
  - `callback.py` - tmux hook callbacks

**Configuration System (`src/tvmux/`):**
- `config.py` - Complete configuration management with TOML support
- `api_client.py` - HTTP client for server communication
- `connection.py` - Connection utilities and health checks
- `utils.py` - Common utilities and helper functions

**Recording Engine (`src/tvmux/`):**
- `repair.py` - Cast file repair utilities for handling abrupt terminations

**Process Management (`src/tvmux/proc/`):**
- `bg.py` - Background process management and cleanup

**Data Models (`src/tvmux/models/`):**
- Pydantic models for sessions, windows, panes, positions, and recordings
- Type-safe data structures for the REST API
- `remote.py` - Remote connection models

### Key Architecture Decisions

**Client-Server Pattern:**
- REST API at 127.0.0.1:21590 for tmux integration
- Server manages global state and multiple window recordings
- CLI tools communicate via HTTP to the server

**Active Pane Following:**
- Records only the currently active pane rather than entire sessions
- Dramatically reduces file sizes while capturing relevant workflow
- Automatically switches recording focus when user changes panes

**FIFO-based Streaming:**
- Uses named pipes to stream terminal output to asciinema
- Enables real-time recording with proper terminal state preservation
- Handles pane switching by dumping state and redirecting streams

## Recording Flow

1. Server starts and sets up tmux hooks for pane change callbacks
2. When recording starts, creates FIFO and launches asciinema process
3. Dumps initial pane state (content + cursor position) to FIFO
4. Starts `tmux pipe-pane` to stream live output to FIFO
5. On pane switches, stops old stream, dumps new state, starts new stream
6. On stop, sends terminal reset sequences and repairs cast file

## Output Organization

Recordings are organized by date: `output_dir/YYYY-MM/timestamp_hostname_session_window.cast`

## Configuration

tvmux supports flexible configuration through multiple sources:

**Configuration Files:**
- `~/.tvmux.conf` - TOML format user configuration
- `example.tvmux.conf` - Example configuration template

**Environment Variables:**
- `TVMUX_OUTPUT_DIR` - Override output directory
- `TVMUX_SERVER_PORT` - Override server port (default: 21590)
- `TVMUX_CONFIG_FILE` - Custom config file location
- `TVMUX_AUTO_START` - Enable/disable auto-start server

**Configuration Sections:**
- `[output]` - Recording output settings (directory, date format)
- `[server]` - Server settings (port, auto-start, auto-shutdown)
- `[recording]` - Recording behavior (repair, pane following)
- `[annotations]` - Annotation options (cursor state)

## Development Notes

- Python 3.11+ with type hints and Pydantic models (tested on 3.13.3)
- Async/await for server operations
- Uses `make test` (pytest) for testing
- Ruff for linting and formatting (120 char line length)
- Pre-commit hooks enforce code quality
- Background process management prevents orphaned processes
- Full configuration system with TOML support and environment overrides

**IMPORTANT:** Do not run the TUI application (`tvmux` or `python -m tvmux.tui.app`) in development - it will interfere with the terminal session.

- you can use tmux capture-pane in this project. The user allows it.
- logs are in /tmp/tvmux-$(whoami)

---
> Source: [bitplane/tvmux](https://github.com/bitplane/tvmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
