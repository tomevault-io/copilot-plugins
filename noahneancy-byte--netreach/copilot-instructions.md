## netreach

> NetReach (no-login fork) — Python CLI + library that gives AI agents read/search access to 7 no-login, low-risk internet sources.

# CLAUDE.md

## Project
NetReach (no-login fork) — Python CLI + library that gives AI agents read/search access to 7 no-login, low-risk internet sources.
Positioning: installer + doctor + config tool. NOT a wrapper — after install, agents call upstream tools directly.
This fork removes all login/Cookie channels (Twitter, Reddit, Facebook, Instagram, XiaoHongShu, LinkedIn, Xueqiu, Xiaoyuzhou).
License: MIT | Version: 2.0.0

## Commands
- `pip install -e .` — Dev install
- `pytest tests/ -v` — All tests
- `pytest tests/test_cli.py -v` — CLI tests only
- `python -m agent_reach.cli doctor` — Run diagnostics
- `python -m agent_reach.cli install --env=auto` — Auto-configure

## Structure
- `agent_reach/cli.py` — CLI entry point (argparse; install / doctor / version)
- `agent_reach/core.py` — Core read/search routing logic
- `agent_reach/config.py` — Config management (YAML, env vars; proxy only)
- `agent_reach/doctor.py` — Diagnostics engine
- `agent_reach/probe.py` — Upstream command probing (missing / broken / timeout / ok)
- `agent_reach/channels/` — One file per platform (web.py, youtube.py, rss.py, github.py, v2ex.py, bilibili.py, exa_search.py)
- `agent_reach/channels/base.py` — Base channel class
- `agent_reach/channels/mcporter.py` — Read-only mcporter config helpers
- `agent_reach/skill/` — SKILL.md + references/ (search / web / dev / video)
- `agent_reach/utils/` — paths / process / text / url helpers
- `tests/` — pytest tests

## Conventions
- Python 3.10+ with type hints
- Each channel is a single file in `channels/`, inherits from `Channel`
- Channel contract: implement `can_handle(url)` and `check(config)`; `backends` is an ordered candidate list
- `check()` must really probe backends (via `probe_command`) and set `self.active_backend`
- Use `loguru` for logging, `rich` for CLI output
- Commit format: `type(scope): message` (one commit = one thing)
- All upstream tool calls go through public API/CLI, never hack internals

## Rules
- NEVER modify upstream open source projects' source code
- NetReach is a "glue layer" — only route and call, don't reimagine
- No login/Cookie channels in this fork — never re-add credential-based platforms
- Version in THREE places must match: `pyproject.toml`, `__init__.py`, `tests/test_cli.py`
- Always new branch for changes, PR to main, never push to main directly
- Run `pytest tests/ -v` before committing — all tests must pass

---
> Source: [noahneancy-byte/netreach](https://github.com/noahneancy-byte/netreach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
