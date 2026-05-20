## python-apple-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Testing Commands
- Install dependencies: `pip install -r requirements.txt`
- Install test dependencies: `pip install -r requirements-test.txt`
- Run all tests: `pytest`
- Run single test file: `pytest tests/test_file.py`
- Run specific test: `pytest tests/test_file.py::test_function`
- Run tests with coverage: `pytest --cov=utils tests/`

## Code Style Guidelines
- Use Python 3.9+ features and syntax
- Follow PEP 8 naming conventions (snake_case for functions/variables, PascalCase for classes)
- Use type hints with proper imports from `typing` module
- Import order: standard library, third-party, local modules
- Use proper exception handling with specific exception types
- Document classes and functions with docstrings using the Google style
- Define models using Pydantic for data validation
- Use asynchronous functions (async/await) for AppleScript operations
- Log errors and debug information using the logging module

---
> Source: [jxnl/python-apple-mcp](https://github.com/jxnl/python-apple-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
