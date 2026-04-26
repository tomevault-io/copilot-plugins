## dkjason

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

dkjason is a Python library that provides JSON serialization capabilities with special support for Django QuerySets, ttcal objects, and custom serialization via `__json__()` methods. The name's intentional misspelling ("jason" instead of "json") is a playful touch.

## Key Dependencies

- Django (version varies by Python version, see requirements.txt)
- ttcal (>=1.0.5) - Time/calendar utilities
- numpy (optional, for numpy.int64 serialization support)

## Development Commands

### Testing

This project uses a dedicated virtualenv located at `/srv/venv/dkjason311` for testing.

**Important**: Always clear the DJANGO_SETTINGS_MODULE environment variable before running tests to avoid conflicts with the test configuration.

```bash
# For Git Bash/WSL/Unix-like environments:
# Clear DJANGO_SETTINGS_MODULE and run all tests with coverage
DJANGO_SETTINGS_MODULE= c:/srv/venv/dkjason311/Scripts/pytest -v --cov=dkjason tests

# Clear and run a specific test file
DJANGO_SETTINGS_MODULE= c:/srv/venv/dkjason311/Scripts/pytest tests/test_jason.py -v

# Clear and run with coverage report (shows missing lines)
DJANGO_SETTINGS_MODULE= c:/srv/venv/dkjason311/Scripts/pytest --cov=dkjason --cov-report=term-missing tests

# Clear and run a single test
DJANGO_SETTINGS_MODULE= c:/srv/venv/dkjason311/Scripts/pytest tests/test_jason.py::test_function_name -v

# For Windows Command Prompt:
# Use "set DJANGO_SETTINGS_MODULE= &&" before each command instead
```

### Building and Installation
```bash
# Install development dependencies
pip install -r requirements.txt

# Install package in development mode
pip install -e .

# Build distribution packages
python setup.py sdist bdist_wheel
```

### Publishing
```bash
# Use invoke to publish (configured in tasks.py)
inv publish
```

## Code Architecture

### Core Module Structure
- `dkjason/jason.py`: Main serialization logic with `DkJSONEncoder` class
  - Handles special types: Decimal, datetime, QuerySet, ttcal objects
  - Supports custom serialization via `__json__()` methods
  - Provides `dumps()`, `response()`, and `r()` functions for JSON responses

### Key Functions in jason.py
- `DkJSONEncoder`: Custom JSON encoder handling special cases
- `dumps()`: Serialize Python objects to JSON string
- `response()`: Create Django HttpResponse with JSON content
- `r()`: Shorthand for `response()`
- `loads()`: Deserialize JSON with special type handling

### Special Serialization Format
Objects are encoded with `@type:value` format for special types:
- `@datetime:` for datetime objects
- `@date:` for date objects
- `@duration:` for ttcal.Duration objects

## Testing Configuration

Tests use Django's test framework with SQLite database configured in `tests/conftest.py`.

**Important Requirements**:
1. All tests must be run using the dedicated virtualenv at `/srv/venv/dkjason311` to ensure proper dependencies and Python version.
2. The DJANGO_SETTINGS_MODULE environment variable must be cleared before running tests, as the tests configure their own Django settings in `conftest.py` and any existing settings module could cause conflicts.

The test suite includes:
- `test_jason.py`: Core serialization tests
- `test_dkjason_import.py`: Import verification

## Windows-Specific Notes

This codebase is being developed on Windows. When working with paths:
- Use forward slashes (`/`) in DOS pathnames
- The backslash (`\`) must be doubled (`\\`) when used in strings that treat it as an escape character

## Version Management

Current version: 3.0.6 (defined in `dkjason/__init__.py` and `setup.py`)

The package supports multiple Python versions with conditional dependencies defined in `requirements.txt` for Django and numpy based on Python version.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/datakortet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
