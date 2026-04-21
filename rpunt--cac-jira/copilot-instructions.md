## cac-jira

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

cac-jira is a CLI tool for interacting with Jira, built on the `cac-core` framework. It follows a `jira <command> <action> [options]` pattern (e.g., `jira issue list`, `jira project show`).

## Development Commands

```bash
# Setup
uv sync                        # Install all dependencies
source .venv/bin/activate      # Activate virtual environment

# Testing
uv run pytest                  # Run all tests
uv run pytest tests/test_cli.py  # Run a single test file

# Linting & Formatting
ruff check --target-version=py39 .   # Lint
ruff format --diff .                  # Check formatting

# Type checking
mypy cac_jira/
```

## Architecture

### Command-Action Discovery

Commands and actions are **dynamically discovered** at runtime by scanning the filesystem — no registration needed. The CLI entry point (`cac_jira/cli/main.py`) scans `cac_jira/commands/` for subdirectories (commands) and their Python modules (actions).

### Class Naming Convention

Action classes **must** follow the pattern `{Command}{Action}` with each part capitalized. For example, a file at `commands/issue/create.py` must define a class named `IssueCreate`. The discovery system in `main.py` uses `f"{command.capitalize()}{action.capitalize()}"` to find the class.

### Class Hierarchy

```
Command (cac_core)
└── JiraCommand (cac_jira/commands/command.py)
    ├── JiraIssueCommand (cac_jira/commands/issue/__init__.py)
    │   └── Individual actions (create, list, show, update, begin, close, etc.)
    └── JiraProjectCommand (cac_jira/commands/project/__init__.py)
        └── Individual actions (list, show)
```

Each action must implement two abstract methods: `define_arguments(parser)` and `execute(args)`.

### Module Initialization

`cac_jira/__init__.py` runs at import time and handles first-run setup (prompting for server, username, project) and creates three global singletons: `JIRA_CLIENT`, `CONFIG`, and `log`. These are imported by `JiraCommand` and available to all actions.

### Key Dependencies

- **cac-core**: Provides base `Command` class, config management (`cac.config.Config`), credential storage (`cac.credentialmanager.CredentialManager`), logging, output formatting (`cac.output.Output`), and update checking.
- **jira**: The `jira-python` library, wrapped by `JiraClient` (`cac_jira/core/client.py`).

### Adding a New Action

1. Create a new `.py` file in the appropriate command directory (e.g., `cac_jira/commands/issue/myaction.py`)
2. Define a class following the naming convention (e.g., `IssueMyaction`)
3. Inherit from the command's base class (e.g., `JiraIssueCommand`)
4. Implement `define_arguments()` and `execute()`

### Configuration & Credentials

- Config file: `~/.config/cac_jira/config.yaml`
- Config template: `cac_jira/config/cac_jira.yaml`
- Credentials stored in system keyring (e.g., macOS Keychain) under `cac-jira`

## Code Style

- Python >=3.10 required (CI tests 3.9–3.12)
- Black formatting with 88-char line length
- isort with black-compatible profile
- Output formatting uses `cac_core.output.Output` (supports table and JSON formats)

## CI/CD

- PRs require a version bump in `pyproject.toml` (enforced by `version-check.yaml`)
- Tests run on push and PR across Python 3.9–3.12
- Releases triggered by version tags (v*), published to PyPI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rpunt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
