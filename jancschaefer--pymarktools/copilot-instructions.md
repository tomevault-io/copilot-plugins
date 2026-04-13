## pymarktools

> This guide provides instructions for OpenAI Codex and similar AI agents when working with this repository. pymarktools

# AGENTS.md for pymarktools

This guide provides instructions for OpenAI Codex and similar AI agents when working with this repository. pymarktools
is a Python library that helps validate and refactor markdown files by checking links and images and providing CLI
tools.

## Project Structure

- `src/pymarktools/` – library source code
- `tests/` – pytest suite mirroring the source layout
- `.github/workflows/` – CI configuration

## Style Guide

- Target **Python 3.13** and include type hints for all public functions.
- Use **NumPy‑style** docstrings.
- Follow `snake_case` for functions and variables and `PascalCase` for classes.
- Organize imports into three groups: standard library, third‑party, and local modules.
- Use `uv` for running commands and managing the virtual environment.

## Development Workflow

1. Run commands through `uv`:
   - `uv run ty check` – type checking.
   - `uv run ruff check src/pymarktools tests --fix` – lint and auto-fix issues.
   - `uv run ruff format --check src/pymarktools tests` – ensure formatting.
   - `uv run pytest` – run the full test suite.
   - `uv run pytest --cov=src/pymarktools --cov-fail-under=80` – run tests with coverage and fail if coverage drops
     below 80%.
1. When adding, changing, or removing features, update `CHANGELOG.md` using the existing structure (e.g. `[unreleased]`,
   `Added`, `Changed`, etc.).
1. Tests should live in the `tests/` directory and mock external network calls.
1. Use `uv run python` for all script execution.

## Pull Requests

- Provide a clear description of the changes and link any related issues.
- Ensure that all checks above pass before submitting the PR.
- Verify that test coverage remains above 80%.
- Keep PRs focused and update documentation where appropriate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jancschaefer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
