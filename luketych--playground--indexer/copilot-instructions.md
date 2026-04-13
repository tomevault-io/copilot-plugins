## playground-indexer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a **Playground Organization System** that automatically organizes files by access frequency and themes. It consists of:

### Core Components
- **Backend**: Main organizer logic in `backend/src/playground_organizer/playground_organizer.py`
- **Web UI**: FastAPI-based read-only interface in `backend/start_web_server.py`
- **Frontend**: HTML/CSS/JS static files in `frontend/`
- **CLI Entry Point**: `playground-organizer.py` (wrapper script)

### Key Architecture Patterns
- **Dual organization modes**: Time-based (current/recent/old/archive) and theme-based (AI, productivity, stocks, etc.)
- **Symlink-based organization**: Preserves original structure while creating organized views
- **Read-only web interface**: Web UI never modifies files, only analyzes and previews
- **Environment-driven configuration**: Uses `.env` files and environment variables

## Development Commands

### Setup and Dependencies
```bash
# Install dependencies
uv sync

# Install development dependencies
uv sync --dev
```

### Running the Application
```bash
# CLI - Generate report
uv run playground-organizer --report

# CLI - Preview organization (dry run)
uv run playground-organizer --organize

# CLI - Execute organization
uv run playground-organizer --organize --execute

# CLI - Theme organization
uv run playground-organizer --theme --execute

# Web UI - Read-only interface
uv run web-ui
# or
uv run python backend/start_web_server.py

# Web UI - Debug mode
uv run web-ui-debug
# or
uv run python backend/start_web_server_debug.py
```

### Testing
```bash
# Run all tests
uv run python tests/run_tests.py

# Run specific test suites
uv run python tests/run_tests.py --unit
uv run python tests/run_tests.py --integration
uv run python tests/run_tests.py --e2e

# Run tests with pytest directly
uv run pytest
```

## Configuration

### Environment Variables
Copy `.env.example` to `.env` and customize:
- `WEB_SERVER_HOST/PORT`: Web server configuration
- `CURRENT_THRESHOLD/RECENT_THRESHOLD/OLD_THRESHOLD`: Time-based organization thresholds (days)
- `EXCLUDE_DIRS`: Comma-separated list of directories to exclude
- `PLAYGROUND_CONFIG_FILE`: JSON config file name (default: `.playground-config.json`)

### Runtime Configuration
The system creates a `.playground-config.json` file with:
- Theme mappings (AI, productivity, stocks, etc.)
- Organization modes and symlink settings
- Excluded directories and tracking preferences

## Key File Locations

### Main Entry Points
- `playground-organizer.py`: CLI wrapper script
- `backend/start_web_server.py`: Web UI server
- `backend/start_web_server_debug.py`: Debug web server

### Core Logic
- `backend/src/playground_organizer/playground_organizer.py`: Main organizer class (465 lines)
- `frontend/templates/index.html`: Web UI template (embedded in Python)
- `frontend/static/`: CSS and JavaScript files (embedded in Python)

### Configuration & Scripts
- `.env.example`: Environment variable template
- `scripts/`: Shell scripts for automation (analyze-access.sh, auto-organize.sh, file-watcher.sh, quick-analyze.sh)
- `tests/run_tests.py`: Test runner with multiple test suites

## Development Notes

### Web UI Architecture
- **FastAPI backend** serves both API and static files
- **Read-only by design**: All `/api/organize` endpoints run in dry-run mode
- **Dynamic file creation**: Templates and static files are generated in Python code
- **Theme detection**: Automatic categorization based on directory names

### CLI Architecture
- **Dual organization**: Time-based and theme-based modes can run together
- **Symlink preservation**: Original files remain in place, organized views use symlinks
- **Safe by default**: All operations run in dry-run mode unless `--execute` is specified

### Testing Strategy
- **Unit tests**: Core functionality testing
- **Integration tests**: Component interaction testing
- **End-to-end tests**: Full workflow testing
- **Test playground**: `tests/test_playground/` with sample directory structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luketych)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/luketych)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
