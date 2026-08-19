## weblens

> WebLens — Python CLI + library that gives AI agents read/search access to 8 internet platforms (all zero-config, no login, no API key).

# CLAUDE.md

## Project
WebLens — Python CLI + library that gives AI agents read/search access to 8 internet platforms (all zero-config, no login, no API key).
Positioning: installer + doctor + config tool. NOT a wrapper — after install, agents call upstream tools directly.
License: MIT | Version: 0.1.0

## Commands
- `pip install -e .` — Dev install
- `pytest tests/ -v` — All tests
- `bash test.sh` — Full integration test (creates venv, installs, runs doctor + channel tests)
- `python -m weblens.cli doctor` — Run diagnostics
- `python -m weblens.cli install --env=auto` — Auto-configure

## Structure
- `weblens/cli.py` — CLI entry point (argparse)
- `weblens/core.py` — Core read/search routing logic
- `weblens/config.py` — Config management (YAML, env vars)
- `weblens/doctor.py` — Diagnostics engine
- `weblens/channels/` — One file per platform (web.py, youtube.py, bilibili.py, etc.)
- `weblens/channels/base.py` — Base channel class (all channels inherit from this)
- `weblens/integrations/mcp_server.py` — MCP server integration
- `weblens/skill/` — Agent skill files
- `tests/` — pytest tests
- `config/mcporter.json` — MCP tool config

## Conventions
- Python 3.10+ with type hints
- Each channel is a single file in `channels/`, inherits from `Channel`
- Channel contract: must implement `can_handle(url)` and `check()` methods
- Use `loguru` for logging, `rich` for CLI output
- Commit format: `type(scope): message` (one commit = one thing)
- All upstream tool calls go through public API/CLI, never hack internals

## Rules
- NEVER modify upstream open source projects' source code
- WebLens is a "glue layer" — only route and call, don't reimagine
- Version in THREE places must match: `pyproject.toml`, `__init__.py`, `tests/test_cli.py`
- Run `pytest tests/ -v` before committing — all tests must pass

---
> Source: [Nontylee/weblens](https://github.com/Nontylee/weblens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
