## starbattlecv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- Test all: `make test` or `uv run python -m pytest --doctest-modules`
- Test single: `uv run python -m pytest tests/test_foo.py::test_function_name`
- Lint/check: `make check` or `uv run pre-commit run -a`
- Type check: `uv run mypy`
- Install deps: `uv sync`

## Code Style

- Type annotations: Required for all functions (mypy with strict settings)
- Line length: 120 characters
- Python version: 3.12+ required
- Formatting: Ruff
- Docstrings: Google-style with Args/Returns sections
- Naming: snake_case for functions/variables, PEP 8 conventions
- Imports: Sorted using isort rules (Ruff "I" selectors)
- Error handling: Follow Tryceratops conventions (Ruff "TRY" selectors)

## Project Structure

- Source code in `starbattlecv/` directory
- Tests in `tests/` directory with test files prefixed with `test_`
- Run tests across all supported Python versions with `tox`
- Example images for use in `examples/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/umbrae)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/umbrae)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
