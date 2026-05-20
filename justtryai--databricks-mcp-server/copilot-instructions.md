## environment

> Development Environment

# Development Environment Standards

## Python Requirements
- Python version: >= 3.10
- Package manager: uv
- Virtual environment: .venv

## Project Configuration
- Use pyproject.toml for dependencies and build configuration
- Recommended IDE: Cursor
- Linting:
  - pylint
  - flake8
  - mypy for type checking

## Getting Started
```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows

# Install dependencies with uv
uv pip install -e .
```

## Running Tests
```bash
# Install development dependencies
uv pip install -e ".[dev]"

# Run tests with pytest
pytest tests/
```

## Linting
```bash
# Run all linters
pylint src/ tests/
flake8 src/ tests/
mypy src/
```

---
> Source: [JustTryAI/databricks-mcp-server](https://github.com/JustTryAI/databricks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
