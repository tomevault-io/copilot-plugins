## rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python RAG (Retrieval-Augmented Generation) project in early stages of development, using uv for dependency management.

## Development Commands

### Environment Setup
```bash
# Install dependencies
uv sync
```

### Running Code
```bash
# Run the main application
uv run python main.py

# Run test script
uv run python test.py
```

### Linting
```bash
# Run ruff for linting and formatting
uv run ruff check .
uv run ruff format .
```

## Project Structure

- `main.py` - Main application entry point
- `test.py` - Test/experimentation script
- `pyproject.toml` - Project dependencies and configuration managed by uv

## Technical Details

- Python version: 3.14+
- Package manager: uv
- Key dependencies: requests, ruff

---
> Source: [ramvarra/rag](https://github.com/ramvarra/rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
