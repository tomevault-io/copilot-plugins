## google-slidebot

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

A CLI tool that extracts links from Google Slides presentations and posts them to Zoom chat via a TUI interface. Used during live presentations to share relevant links with attendees.

## Architecture

- **CLI** (`cli.py`) - Entry point, orchestrates OAuth, fetches slides, connects to Zoom, launches TUI
- **Slides** (`slides.py`) - Google OAuth + Slides API integration, extracts links from presentations
- **Zoom Chat** (`zoom_chat.py`) - Playwright CDP connection to Chrome, sends messages to Zoom web client
- **TUI** (`tui.py`) - Textual app with slide list and link preview screens
- **Config** (`config.py`) - Paths and constants

## Technology Stack

- **Python 3.10+** - Core implementation
- **Click** - CLI framework
- **Textual** - Terminal UI framework
- **Playwright** - Chrome DevTools Protocol for Zoom interaction
- **google-api-python-client** - Google Slides API
- **keyring** - Secure OAuth token storage
- **uv** - Package management and virtual environments
- **pytest** - Testing framework

## Directory Structure

```
google-slidebot/
├── src/google_slidebot/
│   ├── __init__.py
│   ├── cli.py               # CLI entry point
│   ├── config.py            # Configuration constants
│   ├── slides.py            # Google Slides API integration
│   ├── tui.py               # Textual TUI screens
│   └── zoom_chat.py         # Zoom chat via CDP
├── tests/
│   ├── conftest.py          # Shared fixtures
│   ├── test_cli.py
│   ├── test_config.py
│   ├── test_slides.py
│   ├── test_tui.py
│   └── test_zoom_chat.py
├── pyproject.toml           # Project configuration
├── Makefile                 # Development commands
└── CLAUDE.md                # This file
```

## Development Commands

```bash
make help          # Show all available commands
make dev-setup     # Complete development setup
make test          # Run tests
make lint          # Run linting
make format        # Format code
make check         # Run lint + format + test
```

## Testing

```bash
make test                    # Run all tests
make quick-test              # Run tests, stop on first failure
make test-coverage           # Run with coverage report
uv run pytest -v             # Run with verbose output
```

## Development Workflow

1. Write tests first (TDD)
2. Implement minimal code to pass
3. Run `make check` before committing

## Key Implementation Notes

- Zoom web client runs in an iframe; JavaScript injection navigates the iframe DOM
- OAuth tokens stored in system keychain via `keyring` library
- Unicode characters (curly quotes, etc.) normalized to ASCII for Zoom compatibility
- ListView selection uses `on_list_view_selected` event, not key bindings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dannyob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
