## sn2md

> This file provides guidance when working with code in this repository.

# Agent Guidelines

This file provides guidance when working with code in this repository.

## Project Overview

See this projects README.md for an overview of the project, and ADR documents in docs/adr/ for architectural decisions.

## Development Commands

**IMPORTANT**: Always use the Makefile commands for development tasks.

### Setup
```bash
make setup  # Set up venv and sync dependencies with uv
```

### Testing
```bash
make test                                    # Run all tests with pytest and coverage
uv run pytest tests/test_foo.py             # Run specific test file
uv run pytest tests/test_foo.py::test_name  # Run specific test
uv run pytest -v                             # Verbose output
```

### Code Quality
```bash
make lint   # Check code with ruff
make fix    # Auto-fix ruff issues
make type   # Type check with mypy (strict mode)
make radon  # Check cyclomatic complexity and maintainability
make ci     # Run all CI checks (lint + type + test + radon)
```

**Before completing any feature**, run `make ci` to ensure all checks pass.

## Code Quality Requirements

Writing guidance:

- Only include a brief description of the thing in docstrings (no arguments or return types)
- Do not use docstrings at the beginning of files.

## Project Structure

```
project_alpha/       # Main package source code
tests/          # Test files (mirror treepeat/ structure)
docs/adr/       # Architecture Decision Records
```

---
> Source: [dsummersl/sn2md](https://github.com/dsummersl/sn2md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
