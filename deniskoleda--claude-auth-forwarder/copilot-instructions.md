## claude-auth-forwarder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Claude Auth Forwarder** — Telegram bot that monitors a Gmail inbox for authentication emails from Claude.ai/Anthropic and forwards magic links or verification codes to specified Telegram users. Written in Python 3.11+, runs via asyncio event loop.

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt   # includes production deps

# Run the bot
python main.py

# Linting & formatting
ruff check .            # lint
ruff check --fix .      # lint with auto-fix
ruff format .           # format
ruff format --check .   # check formatting without changes

# Type checking
mypy . --ignore-missing-imports

# Security scanning
bandit -r . -x ./venv,./tests,./.git
safety check -r requirements.txt

# Docker
docker-compose up -d
docker-compose logs -f
```

## Architecture

The bot follows a simple polling loop architecture with three main modules:

- **`main.py`** — Entry point. Validates config, authenticates Gmail, sends Telegram startup message, then runs an infinite polling loop: fetch unread emails → send to Telegram → mark as read. Errors trigger a 30-second backoff.

- **`gmail_monitor.py`** — Gmail API integration via OAuth 2.0. Handles token refresh/re-auth, email fetching by configurable query, and extraction of auth data (desktop/mobile magic links, 6-digit codes) using regex patterns. Supports both plain text and HTML email bodies.

- **`telegram_bot.py`** — Sends formatted messages to all `ALLOWED_USER_IDS` via `python-telegram-bot`. Different message templates for desktop links, mobile links, verification codes, and extraction failures.

- **`i18n.py`** — Translation system (EN/RU). All user-facing strings go through `t(key, **kwargs)`.

- **`config.py`** — Module-level constants imported directly (not a class). Created by copying `config.example.py`. Gitignored.

## Key Conventions

- **Config as module**: settings are plain module-level variables in `config.py`, imported as `config.SETTING_NAME`
- **No test suite**: CI runs linting (ruff), type checking (mypy), and security scanning (bandit/safety) but there are no unit tests
- **Ruff rules**: E, W, F, I, B, C4, UP, SIM enabled; line length 100; E501 ignored (handled by formatter)
- **Mypy**: strict mode with `ignore_missing_imports = true`
- **Bandit**: B101 (assert_used) skipped
- **OAuth tokens**: stored in `data/token.json`; `credentials.json` at project root — both gitignored
- **Docker**: runs as non-root `botuser`, port 8080 exposed for OAuth redirect flow

---
> Source: [DenisKoleda/claude-auth-forwarder](https://github.com/DenisKoleda/claude-auth-forwarder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
