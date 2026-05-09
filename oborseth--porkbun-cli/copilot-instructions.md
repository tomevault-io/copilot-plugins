## porkbun-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Porkbun CLI is a Python CLI tool for managing domains and DNS records via the Porkbun API. It uses Typer for CLI structure, Rich for terminal output, Pydantic for config validation, and Requests for HTTP.

## Commands

**Setup:**
```bash
python3 -m venv venv && source venv/bin/activate
pip install -e .
pip install -r requirements-dev.txt
```

**Run:**
```bash
porkbun --help
python3 -m porkbun_cli.cli [command]
```

**Test:**
```bash
pytest
pytest --cov=porkbun_cli
```

**Lint/Format:**
```bash
black porkbun_cli/
mypy porkbun_cli/
flake8 porkbun_cli/
```

## Architecture

**Entry point:** `porkbun_cli/cli.py` — registers 7 Typer subapps and top-level commands (`ping`, `pricing`).

**Core layers:**
- `api.py` — `PorkbunClient` wraps all Porkbun API endpoints. A single `_request()` method handles HTTP (POST) and raises `PorkbunAPIError` on failure. Credentials are injected per-request.
- `config.py` — `ConfigManager` loads/saves `~/.porkbun/config.json` (permissions 0o600). `Config` is a Pydantic model.
- `utils.py` — shared Rich console, output helpers (`print_success`, `print_error`, etc.), `create_table()`, and prompt utilities.
- `commands/` — one module per feature area (dns, domain, ssl, forward, glue, dnssec, config). Each module defines a Typer subapp and a `get_client()` factory that loads credentials and returns a `PorkbunClient`.

**Data flow:** CLI command → `get_client()` → `PorkbunClient` method → `_request()` → Porkbun API → format with Rich utilities → print.

## Conventions

**Adding a command:** create or extend a module in `commands/`, use `get_client()` to get the client, catch `PorkbunAPIError` and call `print_error()` then `raise typer.Exit(1)`. Register a new module in `cli.py` via `app.add_typer()`.

**Destructive commands** must accept a `--yes` flag to skip the confirmation prompt (`confirm()` from `utils.py`).

**Optional parameters** that are likely to be missing in non-interactive use should prompt via `prompt_string()` / `prompt_int()` / `prompt_choice()` rather than hard-failing.

---
> Source: [oborseth/Porkbun-CLI](https://github.com/oborseth/Porkbun-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
