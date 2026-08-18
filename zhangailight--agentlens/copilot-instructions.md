## agentlens

> AgentLens — Python CLI + library that gives AI agents read/search access to 5 internet platforms via official/public channels.

# CLAUDE.md

## Project
AgentLens — Python CLI + library that gives AI agents read/search access to 5 internet platforms via official/public channels.
Positioning: installer + doctor + config tool. NOT a wrapper — after install, agents call upstream tools directly.
Repo: github.com/zhangailight/agentlens | License: MIT | Version: 0.1.0

## Commands
- `pip install -e .` — Dev install
- `pytest tests/ -v` — All tests
- `pytest tests/test_cli.py -v` — CLI tests only
- `bash test.sh` — Full integration test (creates venv, installs, runs doctor + channel tests)
- `python -m agentlens.cli doctor` — Run diagnostics
- `python -m agentlens.cli install --env=auto` — Auto-configure

## Structure
- `agentlens/cli.py` — CLI entry point (argparse)
- `agentlens/core.py` — Core read/search routing logic
- `agentlens/config.py` — Config management (YAML, env vars)
- `agentlens/doctor.py` — Diagnostics engine
- `agentlens/channels/` — One file per platform (github.py, v2ex.py, rss.py, web.py, exa_search.py)
- `agentlens/channels/base.py` — Base channel class (all channels inherit from this)
- `agentlens/integrations/mcp_server.py` — MCP server integration
- `agentlens/skill/` — Agent skill files
- `agentlens/guides/` — Usage guides
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
- AgentLens is a "glue layer" — only route and call, don't reimagine
- Version in `pyproject.toml` and `__init__.py` must match
- Always new branch for changes, PR to main, never push to main directly
- Run `pytest tests/ -v` before committing — all tests must pass

---
> Source: [zhangailight/agentlens](https://github.com/zhangailight/agentlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
