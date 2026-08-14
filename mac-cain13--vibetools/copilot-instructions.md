## vibetools

> A Python CLI tool for managing git worktrees with remote SSH development support. Built with Typer and Rich.

# Vibe - Git Worktree Manager

## Project Overview

A Python CLI tool for managing git worktrees with remote SSH development support. Built with Typer and Rich.

## Setup

Install in development mode:
```bash
pip install -e ".[dev]"
```

Or use the install script (requires `uv`):
```bash
./install.sh
```

## Development Commands

### Run Tests (ALWAYS do this before completing work)
```bash
python3 -m pytest tests/ -v
```

### Run Tests with Coverage
```bash
python3 -m pytest tests/ -v --cov=vibe --cov-report=term-missing
```

### Run Specific Test File
```bash
python3 -m pytest tests/test_cli_integration.py -v
```

### Run the CLI During Development
```bash
python3 -m vibe --help
python3 -m vibe                         # Connect to current repo/worktree (prompts for tool)
python3 -m vibe feature-branch --codex  # Use Codex
python3 -m vibe feature-branch --claude # Use Claude Code
python3 -m vibe feature-branch --oc     # Use OpenCode
python3 -m vibe resume BZL_q7m2x        # Resume parked work from the NSProject board
```

## Project Structure

```
vibe/
├── __init__.py      # Package init
├── __main__.py      # Entry point for python -m vibe
├── cli.py           # Typer CLI interface (incl. 'vibe resume <ticket>')
├── platform.py      # Platform detection (macOS vs WSL)
├── config.py        # Constants (paths, SSH settings) — platform-aware
├── connection.py    # SSH and local connection handling
├── cleanup.py       # Worktree cleanup operations (incl. post-session cleanup)
├── git_ops.py       # Git operations (worktree, branch management)
├── nsproject.py     # NSProject board reader for parked work (find/parse work[], resume write)
└── utils.py         # Shared utilities (console, formatting)

tests/
├── test_cli_integration.py  # CLI integration tests
├── test_cleanup.py          # Cleanup module tests
├── test_connection.py       # Connection module tests
├── test_git_ops.py          # Git operations tests
├── test_nsproject.py        # NSProject board store tests (parse, resolve, mark_resumed)
├── test_platform.py         # Platform detection tests
└── test_resume.py           # 'vibe resume' worktree-recovery + launch tests

docs/                  # Specs (nsproject-park.md — the park/resume store contract)
skills/                # Claude Code skills (the 'park' skill — skills/park)
vm-setup/              # macOS VM setup (tart)
vm-setup-windows/      # Windows/Hyper-V/WSL VM setup
```

Parked work lives on the **NSProject board** (a separate sibling repo), not in a
flat store — `vibe` reads it via `vibe/nsproject.py`; the `park` skill writes it.

## Key Files

- `vibe/platform.py` - Platform detection (macOS vs WSL), override with `VIBE_PLATFORM=wsl`
- `vibe/config.py` - All configurable constants (SSH key path, remote host, worktree paths) — platform-conditional
- `vibe/cli.py` - Main entry point, handles all CLI flags and routing
- `vibe/nsproject.py` - NSProject board reader for parked work (see `docs/nsproject-park.md` for the normative store contract)
- `pyproject.toml` - Project metadata, dependencies, and tool configuration

## Testing Requirements

- **Always run tests** before considering work complete
- Target: all tests passing
- Coverage target: >80%
- Tests use `pytest` with `pytest-cov` for coverage
- Mock external calls (subprocess, file system) in tests

## Code Style

- Python 3.9+ compatible (use `from __future__ import annotations`)
- Type hints on all functions
- Docstrings with Args/Returns sections
- Use `Path` objects for file paths
- Use `shlex.quote()` for shell escaping

---
> Source: [mac-cain13/vibetools](https://github.com/mac-cain13/vibetools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
