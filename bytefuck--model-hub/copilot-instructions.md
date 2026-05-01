## model-hub

> This document provides guidelines for AI agents working on this LLM Gateway project.

# Agent Guidelines for LLM Gateway

This document provides guidelines for AI agents working on this LLM Gateway project.

## Project Overview

A unified LLM interface proxy that standardizes local and remote LLM APIs into OpenAI-compatible HTTP endpoints. Supports OpenAI, Anthropic, Ollama, Azure, and other providers.

## Build / Lint / Test Commands

### Setup
```bash
# Install dependencies (primary: uv)
uv pip install -e ".[dev]"

# Or using pip
pip install -e ".[dev]"
```

### Development
```bash
# Run the server locally
python -m llm_gateway.server

# Or with auto-reload
uvicorn llm_gateway.server:app --reload --port 8000
```

### Testing
```bash
# Run all tests
pytest

# Run a single test file
pytest tests/test_adapters.py -v

# Run a single test
pytest tests/test_adapters.py::test_openai_adapter -v

# Run with coverage
pytest --cov=llm_gateway --cov-report=term-missing
```

### Linting & Formatting
```bash
# Run all checks
ruff check .
ruff check . --fix
ruff format .
mypy llm_gateway

# One command
ruff check . && ruff format . && mypy llm_gateway
```

## Code Style Guidelines

### Imports
- Group imports: stdlib → third-party → local
- Use `from __future__ import annotations` for forward references
- Absolute imports preferred over relative

### Type Hints
- Use type hints for all function signatures and variables
- Use `Optional[T]` or `T | None` for nullable types
- Use generics where appropriate
- Run `mypy` to verify type safety

### Naming Conventions
- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Private members: `_leading_underscore`
- Abstract base classes: `Base` prefix or `ABC` suffix

### Error Handling
- Use custom exceptions in `llm_gateway.exceptions`
- Always catch specific exceptions, never bare `except:`
- Log errors with context using structlog
- Return proper HTTP status codes (400, 401, 429, 500, 502)

### Async Patterns
- Use `async/await` for all I/O operations
- Use `httpx.AsyncClient` for HTTP requests
- Context managers (`async with`) for resource management

### Configuration
- Use Pydantic Settings for config (see `llm_gateway/config.py`)
- Environment variables for secrets (never hardcode)
- Support `.env` files for local development

### Logging
- Use `structlog` for structured logging
- Log at appropriate levels (DEBUG, INFO, WARNING, ERROR)
- Include request IDs for traceability

### Documentation
- Google-style docstrings for all public APIs
- Type hints serve as documentation
- Keep README.md updated with new features

## Testing Guidelines

- Use `pytest` with `pytest-asyncio` for async tests
- Mock external API calls with `respx` or `pytest-httpx`
- Test both success and error cases
- Use fixtures for common test setup
- Aim for >80% coverage

## Project Structure

```
llm_gateway/
├── adapters/       # Provider-specific adapters
├── models/         # Pydantic models
├── exceptions.py   # Custom exceptions
├── config.py       # Configuration management
└── server.py       # FastAPI application
```

---
> Source: [bytefuck/model-hub](https://github.com/bytefuck/model-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
