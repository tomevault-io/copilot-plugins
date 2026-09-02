## app-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Python package of document-related tools (conversion, processing) exposed to AI assistants through an MCP server (`main.py`, built on `mcp.server.fastmcp.FastMCP`).

## Commands

```bash
# Create and activate a virtual env (Windows: uv adds a --native-tls/--system-certs flag may be
# needed here if uv can't validate PyPI's cert chain — see project memory)
uv venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# Install the package in development mode
uv pip install -e .

# Start the MCP server
uv run main.py

# Run all tests
uv run pytest

# Run a single test file / test
uv run pytest tests/test_document.py
uv run pytest tests/test_document.py::TestBinaryDocumentToMarkdown::test_binary_document_to_markdown_with_docx
```

## Architecture

- **`tools/`** — plain Python functions implementing each tool's logic (e.g. `tools/document.py`, `tools/math.py`). These are independent of MCP and can be unit tested directly, as in `tests/test_document.py`.
- **`main.py`** — the MCP server entrypoint. It imports tool functions and explicitly registers each one with the `FastMCP` instance via `mcp.tool()(function)`. **A function existing in `tools/` does not mean it's exposed** — check `main.py` for the actual registered set (currently only `add` from `tools/math.py`; `binary_document_to_markdown` in `tools/document.py` is implemented but not yet wired up).
- **`tests/`** — pytest tests against the `tools/` functions directly (not through the MCP layer). Fixtures for document-conversion tests live in `tests/fixtures/`.

## Defining a new MCP tool

1. Implement the function in the appropriate module under `tools/` (create a new module if it doesn't fit an existing one).
2. Register it in `main.py`: `mcp.tool()(my_function)`.
3. Follow this shape, using `pydantic.Field` for parameter descriptions:

```python
from pydantic import Field

def my_tool(
    param1: str = Field(description="Detailed description of this parameter"),
    param2: int = Field(description="Explain what this parameter does")
) -> ReturnType:
    """Comprehensive docstring here"""
    # Implementation
```

4. Tool docstrings are the model-facing description and should:
   - Begin with a one-line summary
   - Provide a detailed explanation of functionality
   - Explain when to use (and not use) the tool
   - Include usage examples with expected input/output (see `tools/math.py:add` for the pattern)
5. Type every argument and the return value (as in the example above) — FastMCP derives the tool's input schema from these annotations, so an untyped/loosely-typed arg produces a worse schema for the model calling it.

---
> Source: [kenL707/app_starter](https://github.com/kenL707/app_starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
