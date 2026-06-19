## openclaw-conversation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenClaw Conversation is a Home Assistant custom component that uses OpenClaw as the AI backend for conversation and AI task platforms. It is a simplified fork of `extended_openai_conversation` with OpenClaw-specific features.

## Development Commands

```bash
# Install dependencies
uv sync

# Lint Python code
uv run ruff check custom_components/

# Lint with auto-fix
uv run ruff check --fix custom_components/

# Format Python code
uv run ruff format custom_components/
```

## Git Workflow

- Always use `-s` flag when committing to add DCO (Developer Certificate of Origin) signoff
- Use conventional commit messages: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `perf:`, `test:`, `ci:`, `build:`, `style:`
- Example: `git commit -s -m "feat: add new feature"`

## Architecture

### HTTP Client

Uses Home Assistant's built-in httpx client (no external dependencies):

- `homeassistant.helpers.httpx_client.get_async_client(hass)`
- Calls OpenClaw's `/v1/chat/completions` endpoint with streaming SSE

### Custom Headers

Sends OpenClaw-specific headers with each request:

- `x-openclaw-session-key` - For session routing (optional)
- `x-openclaw-agent-id` - Agent identifier (default: "main")

### Key Differences from extended_openai_conversation

- No OpenAI SDK dependency - uses httpx directly
- No model selection - OpenClaw decides the model
- No temperature/top_p/max_tokens config
- No function/tool execution - OpenClaw handles tools internally
- Entity ID sent as `user` field for session correlation

## Code Style

- Google-style docstrings (enforced by ruff)
- Import aliases: `voluptuous` as `vol`, `config_validation` as `cv`, etc. (see pyproject.toml)
- Use `asyncio.timeout` instead of `async_timeout`
- Use `zoneinfo` instead of `pytz`

## CI Validation

GitHub Actions runs on push to main and PRs:

- `home-assistant/actions/hassfest@master` - validates integration manifest
- `hacs/action@main` - validates HACS requirements

---
> Source: [Djelibeybi/openclaw_conversation](https://github.com/Djelibeybi/openclaw_conversation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
