## open-web-intel-agent

> OpenWebIntel — Python CLI + library that gives AI agents read access to 6 public internet sources.

# CLAUDE.md

## Project
OpenWebIntel — Python CLI + library that gives AI agents read access to 6 public internet sources.
Positioning: installer + doctor + config tool. NOT a wrapper — after install, agents call upstream tools directly.
License: MIT | Version: 1.0.0

## Principles
- Zero login, zero cookies, zero third-party data services — content is fetched directly from the source.
- Web pages are fetched locally via urllib (no reader proxy); no account cookies anywhere.
- Do NOT add channels that require login cookies, browser-session bridging, or third-party data services.

## Commands
- `pip install -e .` — Dev install
- `pytest tests/ -v` — All tests
- `python -m open_web_intel.cli doctor` — Run diagnostics
- `python -m open_web_intel.cli install --env=auto` — Read-only check

## Structure
- `open_web_intel/cli.py` — CLI entry point (argparse)
- `open_web_intel/core.py` — Core routing logic
- `open_web_intel/config.py` — Config management (YAML, env vars)
- `open_web_intel/doctor.py` — Diagnostics engine
- `open_web_intel/channels/` — One file per platform (web.py, youtube.py, github.py, bilibili.py, v2ex.py, rss.py)
- `open_web_intel/channels/base.py` — Base channel class
- `open_web_intel/skill/` — Agent skill files
- `tests/` — pytest tests

## Conventions
- Python 3.10+ with type hints
- Each channel is a single file in `channels/`, inherits from `BaseChannel`
- Channel contract: must implement `can_handle(url)` and `check()` methods
- Use `loguru` for logging, `rich` for CLI output
- Commit format: `type(scope): message` (one commit = one thing)
- All upstream tool calls go through public API/CLI, never hack internals

## Rules
- NEVER add cookie-based auth, browser-session bridging, or third-party data services
- OpenWebIntel is a "glue layer" — only route and call, don't reimplement
- Version in TWO places must match: `pyproject.toml`, `open_web_intel/__init__.py`
- Always new branch for changes, PR to main, never push to main directly
- Run `pytest tests/ -v` before committing — all tests must pass

---
> Source: [MUZI-LYY/open-web-intel-agent](https://github.com/MUZI-LYY/open-web-intel-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
