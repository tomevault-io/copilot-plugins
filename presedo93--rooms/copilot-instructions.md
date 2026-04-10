## rooms

> **Install dependencies**: `uv sync`

# Agent Guidelines for Rooms Trading System

## Commands

**Install dependencies**: `uv sync`
**Run all tests**: `uv run python -m pytest` (when tests are added)
**Run single test**: `uv run python -m pytest path/to/test_file.py::TestClass::test_method`
**Lint code**: `uv run python -m flake8 .` (when configured)
**Format code**: `uv run python -m black . && uv run python -m isort .` (when configured)
**Type check**: `uv run python -m mypy .` (when configured)

## Code Style

### Imports
- Use absolute imports within the workspace
- Group imports: standard library, third-party, local modules
- Use `from __future__ import annotations` for Python 3.13+ features

### Formatting
- Use Black for code formatting with default settings
- Use isort for import sorting with Black compatibility
- Line length: 88 characters (Black default)

### Types
- Use type hints for all function parameters and return values
- Use `typing` module for complex types (Union, Optional, List, etc.)
- Prefer `from __future__ import annotations` to avoid forward reference issues

### Naming
- Functions/variables: snake_case
- Classes: PascalCase
- Constants: UPPER_SNAKE_CASE
- Modules: lowercase, underscore-separated if needed

### Error Handling
- Use custom exception classes inheriting from built-in exceptions
- Provide descriptive error messages with context
- Use logging for debugging information, not print statements

### Testing
- Use pytest as the testing framework
- Test files: `test_*.py` alongside implementation files
- Test classes: `Test*` with descriptive method names
- Use fixtures for common test setup/teardown

### Documentation
- Use docstrings for all public functions/classes
- Follow Google/NumPy docstring format
- Include type information in docstrings when not using type hints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/presedo93)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/presedo93)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
