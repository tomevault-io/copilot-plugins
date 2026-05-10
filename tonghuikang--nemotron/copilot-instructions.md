## nemotron

> For changes involving html files, please use MCP pupeeteer to test.

For changes involving html files, please use MCP pupeeteer to test.

Package Management
- ONLY use uv, NEVER pip
- Installation: uv add package
- Running tools: uv run tool
- Upgrading: uv add --dev package --upgrade-package package
- FORBIDDEN: uv pip install, @latest syntax

Formatting
- Format: uv run --frozen ruff format *.py
- Check: uv run --frozen ruff check *.py
- Fix: uv run --frozen ruff check *.py --fix
- Sort imports: uv run --frozen ruff check --select I *.py --fix
- Type checking: uv run --frozen mypy *.py

To start the server, do `./serve.sh`

---
> Source: [tonghuikang/nemotron](https://github.com/tonghuikang/nemotron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
