## tunebat-mcp

> - Use `uv` for all Python dependency management.

# Cursor Rules for Python & ClickHouse Development

## Dependency Management
- Use `uv` for all Python dependency management.
  - Add libraries with: `uv add <library>`
  - Run scripts with: `uv run <script.py>`
- Manage dependencies exclusively with `pyproject.toml`.
- **Do NOT** create or use `requirements.txt`.

## Python Development Standards
- Use Python 3.12+ type hints (e.g., `list` instead of `List`).
- Always include type annotations (Pyright-compatible).
- Write idiomatic, modern Python.

## ClickHouse Integration
- Assume all database operations use ClickHouse.
- Use the `clickhouse_connect` library for all ClickHouse interactions.
- For SQL-only requests, focus solely on ClickHouse SQL—ignore Python.

## General Guidelines
- You are an expert in Python and ClickHouse.
- Follow best practices for code clarity, maintainability, and performance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/araa47) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
