## worktree-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Worktree Manager is a standalone Python CLI tool for managing git worktrees with Docker Compose isolation. It's designed for developers working on multiple features simultaneously in projects that use Docker for local development.

## Tech Stack

- **Python 3.11+** - Core language
- **Rich** - Terminal output formatting
- **filelock** - Thread-safe file locking for registry
- **uv** - Package manager (recommended)

## Project Structure

```
worktree-manager/
├── pyproject.toml          # Package configuration and dependencies
├── README.md               # User documentation
├── CLAUDE.md               # This file
├── LICENSE                 # MIT license
└── worktree_manager/       # Main package
    ├── __init__.py
    ├── __main__.py         # Entry point for `python -m worktree_manager`
    ├── cli.py              # Argument parsing (argparse)
    ├── commands.py         # CLI command implementations
    ├── config.py           # Configuration management
    ├── docker_ops.py       # Docker/Compose operations
    ├── git_ops.py          # Git operations (worktree, branch, commit)
    ├── ports.py            # Port allocation logic
    ├── registry.py         # Worktree registry (JSON file storage)
    ├── setup_wizard.py     # Interactive setup
    ├── validator.py        # Environment validation
    └── hooks/              # Lifecycle hooks
        ├── __init__.py     # LifecycleHook protocol + LifecycleHookManager registration
        ├── config.py       # Hook configuration
        ├── claude.py       # ClaudeLaunchHook (launch Claude Code on worktree lifecycle)
        └── uv.py           # UvSyncHook (run uv sync on worktree lifecycle)
```

## Key Commands

The CLI is installed as both `worktree-manager` and the short alias `wt`.

A `justfile` wraps the common workflows — prefer these:

```bash
just                # List all recipes
just sync           # uv sync (install dependencies)
just test           # uv run pytest
just lint           # ruff check .
just fix            # ruff check . --fix
just fmt            # ruff format .
just check          # lint + test
just install        # clean cache + uv tool install . --force (installs `wt` globally)
just bump-patch     # bump version + reinstall (also bump-minor / bump-major)
```

Raw equivalents:

```bash
uv sync
uv run wt --help
uv run pytest tests/test_claude_command.py::<test_name>   # run a single test
uv run ruff format .
uv build
uv tool install . --force
```

IMPORTANT: The globally installed `wt` runs from an installed copy, not the source tree. After changing any code you want reflected in the global `wt`, run `just install` (or `uv tool install . --force`).

## Important Patterns

### Data Storage

- **Registry** (`~/.config/worktree-manager/registry.json`): Worktree metadata, ports, paths

The config directory was renamed from the legacy `~/.config/dispatch-guru/`; `config.py` migrates from the old location on load.

### Shell Integration (auto-`cd`)

A subprocess can't change its parent shell's cwd, so `wt create`/`wt close` can't
`cd` the user's shell directly. Instead:

- `wt shell-init` (`commands.shell_init_cmd`) prints a `wt` shell function that users
  `eval`/`source` from their rc file.
- That function runs the real binary with `WT_CD_FILE` (`commands.CD_TARGET_ENV`) set to
  a temp file, then `cd`s to whatever path the binary wrote there.
- `create_worktree_cmd` and `close_worktree` call `commands._emit_cd_target(path)` to write
  the destination (new worktree on create, main repo on close). It's a no-op when
  `WT_CD_FILE` is unset, so `wt` run directly is unaffected.

### Port Allocation

Ports are allocated by worktree index:
- Index 0: Web 8000, DB 5432
- Index 1: Web 8010, DB 5442
- Index N: Web 8000+N*10, DB 5432+N*10

### Safety Checks (Close Worktree)

Before closing a worktree, the system checks:
1. Uncommitted changes
2. Unpushed commits
3. Branch merged to develop

If any check fails, user must use `--force`.

## Configuration Files

### User Config (`~/.config/worktree-manager/config.json`)

```json
{
  "base_branch": "develop",
  "setup_completed": true,
  "ignore_patterns": [],
  "docker_settings": {
    "stream_output": true,
    "auto_build": true
  }
}
```

### Project Config (`.worktree-manager.json`)

Created per-project with `worktree-manager init`.

## Code Style

- **Formatter**: Ruff
- **Quote style**: Single quotes
- **Line length**: 120 characters
- **Type hints**: Encouraged but not strictly enforced

## Testing

Tests live under `tests/` and run with `just test` (or `uv run pytest`). Coverage is currently partial (e.g. `test_claude_command.py`, `test_claude_hook.py`) — add tests alongside these when introducing new behavior.

## Common Tasks

### Adding a New CLI Command

1. Add command function in `commands.py`
2. Add argument parser in `cli.py`
3. Wire up in `cli.py` main function

### Adding a New Hook

1. Create hook class in `hooks/`
2. Register in `hooks/__init__.py`

## Dependencies

Core (required):
- `rich>=13.0.0` - Terminal formatting
- `filelock>=3.0.0` - File locking

Dev (optional):
- `pytest>=8.0.0` - Testing
- `ruff>=0.8.0` - Linting/formatting
- `mypy>=1.13.0` - Type checking

---
> Source: [bellbird-bakery/worktree-manager](https://github.com/bellbird-bakery/worktree-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
