## olol

> - **Package manager**: `uv`

# OLOL Project Guide

## Build & Test Commands
- **Package manager**: `uv`
- **Run tests**: `pytest -v tests/`
- **Single test**: `pytest -v tests/test_file.py::test_function`
- **Coverage**: `pytest --cov=olol tests/`
- **Benchmark**: `pytest tests/bench_ollama.py -v`
- **Lint**: `ruff check .`
- **Format**: `black . && isort .`
- **Type check**: `mypy src/ tests/`
- **Protocol buffer**: `python src/script/oprotob.py`

## Code Style Guidelines
- **Format**: Use black (88 char line length)
- **Imports**: Sort with isort
- **Types**: Strict typing required for all functions
- **Naming**: snake_case for variables/functions, CamelCase for classes
- **Error handling**: Use try/except with specific exceptions and logging
- **Async**: Use asyncio patterns consistently, avoid excessive exec calls
- **Documentation**: Docstrings for public APIs in Google style

## Architecture
OLOL provides gRPC clients and servers for distributed Ollama LLM inference.
Designed for clustering multiple Ollama instances across systems as a single service.
Supports both async/sync communication with persistent connections for optimization.
Currently being restored to full async/sync support for distributed clustering.

---
> Source: [K2/olol](https://github.com/K2/olol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
