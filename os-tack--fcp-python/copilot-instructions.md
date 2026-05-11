## fcp-python

> MCP server for querying and refactoring Python codebases via pylsp (python-lsp-server).

# fcp-python — Python Code Intelligence FCP Server

MCP server for querying and refactoring Python codebases via pylsp (python-lsp-server).
Ported from fcp-rust, using fcp-core Python package.

## Architecture

4-layer: MCP server -> domain (query/mutation dispatch) -> resolver (symbol resolution) -> LSP client (pylsp subprocess)

### Layers

- **lsp/** — LSP client wrapping pylsp subprocess via asyncio (types, transport, client, lifecycle, workspace_edit)
- **resolver/** — Symbol resolution: triple-indexed SymbolIndex, 3-tier pipeline, selector parsing
- **domain/** — Query/mutation dispatch, verb registration, output formatting
- **main.py** — MCP server entry point (4 tools: python, python_query, python_session, python_help)
- **bridge.py** — Slipstream Unix socket bridge (asyncio)

## Commands

```bash
uv run pytest              # run tests
uv run pytest -x           # stop on first failure
uv run ruff check src/     # lint
uv run pyright src/        # type check
```

## Key Patterns

- All LSP communication is async (asyncio.create_subprocess_exec for pylsp)
- Content-Length framing for JSON-RPC over stdin/stdout
- 3-tier symbol resolution: index -> workspace/symbol -> documentSymbol
- Mutations applied via workspace_edit.py (reverse offset order)
- Python-specific selectors: @class (not @struct), @decorator, @module

---
> Source: [os-tack/fcp-python](https://github.com/os-tack/fcp-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
