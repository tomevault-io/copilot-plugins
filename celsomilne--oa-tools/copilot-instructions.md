## oa-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

oa-tools is a set of tools for analysing how Australian politicians speak publicly about and vote on issues. It includes a CLI and an MCP server. Backing data comes from [They Vote For You](https://theyvoteforyou.org.au) and [OpenAustralia](https://www.openaustralia.org.au).

The repo is a Python monorepo with per-package Poetry projects.

## Commands

### Python

Each Python package has its own `pyproject.toml`, `poetry.lock`, and `.venv/` (managed by Poetry with `in-project = true`). Use `poetry run` to run commands — no manual venv activation needed.

```bash
# Install all Python packages (from repo root):
./scripts/poetry_install.sh

# Run all Python tests:
./scripts/poetry_test.sh

# Library (packages/oa-tools)
cd packages/oa-tools
poetry install                                # install deps + create .venv
poetry run pytest tests/ -v                   # run library tests
poetry run pytest tests/test_tvfy.py -v       # single test file
poetry run pytest tests/ -k "test_search"     # tests matching keyword

# CLI app (apps/cli)
cd apps/cli
poetry install                                # installs library as path dep
poetry run pytest tests/ -v                   # run CLI tests
poetry run oa-tools --help                    # run the CLI

# MCP server (apps/mcp)
cd apps/mcp
poetry install                                # installs library as path dep
poetry run pytest tests/ -v                   # run MCP tests
poetry run oa-tools-mcp                       # run MCP server (stdio)
```

### Checking (lint + types + tests)

```bash
./scripts/check.sh                              # run everything: lint, format, types, architecture, tests
ruff check .                                    # Python lint only (from repo root)
ruff format --check .                           # Python format check only (from repo root)
ruff check --fix . && ruff format .             # auto-fix lint + format
cd packages/oa-tools && poetry run mypy src/    # type check single package
```

## Architecture

- Python 3.12, each package managed independently with Poetry
- `packages/oa-tools/` — core library (Python module: `oa_tools`): models, API clients (TVFY + OpenAustralia), display formatting
- `apps/cli/` — `oa-tools` CLI app (Click), depends on `oa_tools` as a path dependency
- `apps/mcp/` — `oa-tools-mcp` MCP server (FastMCP, 12 tools over stdio), depends on `oa_tools` as a path dependency
- Per-package virtual environments (`.venv/` in each package directory, managed by Poetry)
- API keys in `.env` at repo root (loaded via `python-dotenv`): `THEYVOTEFORYOU_API_KEY`, `OPENAUSTRALIA_API_KEY`

### Dependency rules

Dependency direction is always `apps → packages`. Enforced by `tests/test_architecture.py`:
- `oa_tools` (library) never imports from any app package
- Apps never import from each other
- New packages follow the same rule: libraries in `packages/`, runnable apps in `apps/`

### Data Sources

- **They Vote For You API** (`https://theyvoteforyou.org.au/api/v1/`) — JSON REST API for politicians, divisions (votes), and policies. Endpoints: `people.json`, `people/{id}.json`, `divisions.json`, `divisions/{id}.json`, `policies.json`, `policies/{id}.json`. Auth via `?key=` query param.
- **OpenAustralia API** (`https://www.openaustralia.org.au/api/`) — endpoints for representatives, senators, debates, hansard, divisions. Python bindings via `openaustralia` package. Output defaults to JS format; supports `xml`, `php`, `js`.

## Testing

All Python code must have thorough tests. Each package has its own `tests/` directory using `pytest` + `pytest-mock`.

- **No live API calls in tests.** Mock HTTP and API client methods using `pytest-mock` (`mocker.patch`) or `monkeypatch`.
- **API clients accept keys via constructor** so tests never depend on `.env` or environment variables.
- **Shared fixtures** in each package's `tests/conftest.py` — sample model instances and mock API response dicts matching real response shapes.
- **Display tests** use `capsys` to capture stdout output.
- **CLI integration tests** use Click's `CliRunner` to invoke commands in-process.
- **MCP protocol tests** use in-memory transport (`create_client_server_memory_streams`) to test the full MCP request/response cycle without stdio.
- Tests must pass before any implementation is considered complete.

### Integration tests (real APIs)

Integration tests hit the live TVFY and OpenAustralia APIs to verify response shapes. They are **excluded by default** via `addopts = "-m 'not integration'"` so normal `pytest` runs stay fast and offline.

```bash
# Run integration tests only (requires API keys in .env)
cd packages/oa-tools && poetry run pytest -m integration -v
cd apps/mcp && poetry run pytest -m integration -v

# Run everything (unit + integration)
cd packages/oa-tools && poetry run pytest -m "" -v
cd apps/mcp && poetry run pytest -m "" -v
```

Tests skip gracefully when the corresponding API key env var is not set.

## Ticket Workflow

Feature work follows a structured ticket process under the `tickets/` directory.

Each ticket lives in `tickets/NNNN-short-name/` and contains three sequential documents:

1. **`01-problem.md`** — Problem specification: what problem we're solving and why
2. **`02-solution.md`** — Solution specification: the high-level approach and design
3. **`03-implementation.md`** — Implementation specification: exact files, changes, and steps

Each document is written by Claude and reviewed/approved by the user before proceeding to the next. Implementation only begins after all three are approved.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/celsomilne)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/celsomilne)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
