## glm-launch

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

glm-launch is a Python CLI tool that wraps Claude Code (`claude`) with GLM settings (`codex` and `opencode` are not supported). It requires Python 3.13+ and uses Typer for CLI handling. Configuration is driven by environment variables (`GLM_BASE_URL`, `GLM_API_KEY`, `GLM_AUTH_TOKEN`).

## Commands

```bash
# Show top-level help
uv run src/main.py --help

# Show available providers
uv run src/main.py launch --help

# Launch claude (requires GLM_AUTH_TOKEN; GLM_BASE_URL has a default)
uv run src/main.py launch claude

# Launch claude with a specific model
uv run src/main.py launch claude --model "some-model"

# Claude options can omit the provider name
uv run src/main.py --model "some-model" --dry-run

# codex is disabled (Z.ai has no OpenAI Responses API endpoint); it prints a note and exits

# Pass extra args through to the underlying tool
uv run src/main.py launch claude -- --verbose

# Time a request against the configured GLM endpoint
uv run src/main.py bench

# List known Z.ai GLM models (built-in list)
uv run src/main.py models

# Fetch the live model list from the Z.ai API (needs GLM_AUTH_TOKEN)
uv run src/main.py models --remote

# Bootstrap the current shell with GLM env vars (so a plain `claude` uses Z.ai)
eval "$(uv run src/main.py shell)"

# Run tests, lint checks, and a package build
uv run pytest
uv tool run prek run --all-files
uv build
```

## Architecture

Single-module project with entry point at `src/main.py` (the installed `glm-launch` console script calls `cli()`, which defaults to the `claude` provider when no command is given and forwards bare provider options such as `--model` to Claude). Uses Typer with a two-level command structure: `glm launch <provider>`. Providers are also registered at the top level so `glm-launch <provider>` works without the `launch` prefix. Each provider gets its own `@launch_app.command()` with provider-specific setup logic:

- **claude** — Validates constrained settings, sets `ANTHROPIC_BASE_URL`, `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_DEFAULT_*_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL`, `CLAUDE_CODE_EFFORT_LEVEL`, `CLAUDE_CODE_ATTRIBUTION_HEADER`, `CLAUDE_CODE_AUTO_COMPACT_WINDOW`, and `CLAUDE_CODE_MAX_CONTEXT_TOKENS` env vars from GLM settings, then passes the `--model` flag. Falls back to `~/.claude/local/claude` if not on PATH; `--dry-run` works without an installed binary.
- **codex** — Disabled. Z.ai exposes only Anthropic Messages and OpenAI Chat Completions, but current codex requires the OpenAI Responses API (no `/responses` endpoint → 404). The command prints an explanation and exits 1.

All providers exec the underlying binary via `os.execvpe()` for full stdio passthrough.

---
> Source: [jefftriplett/glm-launch](https://github.com/jefftriplett/glm-launch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
