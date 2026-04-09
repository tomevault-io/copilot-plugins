## lets

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "lets" - an AI-powered development workspace setup tool built in Python. It creates isolated git worktrees with auto-generated branch names, sets up tmux sessions, and launches AI coding assistants for specific tasks.

## Architecture

- **Entry point**: `src/lets/cli.py` contains the main Click CLI application
- **Core functionality**: Single-file architecture with all logic in `cli.py`
- **Configuration**: Uses `pyproject.toml` for dependencies and tool configuration
- **Task automation**: Uses `mise.toml` for development task management

## Key Components

- `WorktreeConfig` dataclass: Centralized configuration for worktree and tmux setup
- `Colors` class: Terminal styling utilities for user feedback
- Git operations: Repository detection, branch management, worktree creation
- Tmux integration: Session and window management for development environments
- AI integration: Calls external AI tools (default: claude) for branch name generation

## Development Commands

**Setup and dependencies:**
```bash
mise run install  # Install all dependencies (Python + npm)
```

**Code quality:**
```bash
mise run check       # Run all checks (ruff + pyright)
mise run check:ruff  # Lint with ruff
mise run check:pyright  # Type check with pyright

mise run format      # Format all code (ruff + prettier + rustywind)
mise run format:ruff # Format Python code only
```

**AI assistant:**
```bash
mise run claude  # Launch Claude with skip permissions
```

## Tool Configuration

- **Ruff**: Configured with ALL rules enabled, minimal ignores, Google docstring convention
- **Pyright**: Uses `.venv` virtual environment, Python 3.12, excludes venv from analysis
- **Click**: Main CLI framework with rich terminal output support

## Application Usage

The tool creates isolated development environments:
```bash
lets "Fix authentication bug"  # Auto-generates branch name
lets "Add feature" --session frontend --branch custom-name
```

Key features:
- Git worktree creation with branch conflict resolution
- Tmux session/window management
- Environment file copying (.env, .env.local, etc.)
- AI-generated branch names from task descriptions
- Interactive prompts for existing branches/directories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/collindutter)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/collindutter)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
