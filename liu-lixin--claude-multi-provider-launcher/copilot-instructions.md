## claude-multi-provider-launcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A secure shell script launcher for Claude Code that enables switching between multiple AI API providers (Zhipu/GLM, MiniMax, Moonshot/Kimi, DeepSeek, and Anthropic). The architecture separates configuration from code by storing all sensitive data in `~/.start-claude.env` (gitignored).

## Key Commands

### Setup and Configuration
```bash
# Initial setup or apply configuration changes
./install.sh

# Edit configuration (API keys, models, URLs)
vim ~/.start-claude.env

# Launch Claude Code with provider selection
~/start-claude.sh
```

### Development Workflow
```bash
# After modifying start-claude.sh or install.sh
./install.sh  # Updates symlink and validates changes
~/start-claude.sh  # Test the changes

# Commit changes
git add . && git commit -m "description"
```

## Architecture

### Configuration Flow
1. User configuration lives in `~/.start-claude.env` (created from `start-claude.env.example`)
2. `start-claude.sh` loads the env file and presents a provider selection menu
3. Selected provider's settings are written to `~/.claude/settings.json` using `jq`
4. Claude CLI is launched with the configured provider

### Key Files
- `start-claude.sh` - Main launcher (symlinked to `~/start-claude.sh`)
- `install.sh` - Setup script (creates symlink, validates config, installs dependencies)
- `start-claude.env.example` - Template (committed to git)
- `~/.start-claude.env` - User config (gitignored, contains actual API keys)
- `~/.claude/settings.json` - Claude Code settings (modified by launcher)
- `~/.claude.json` - Onboarding flag (created by install script)

### Provider Configuration Pattern
Each provider requires:
- `{PROVIDER}_API_KEY` - Authentication token
- `{PROVIDER}_BASE_URL` - API endpoint
- `{PROVIDER}_OPUS_MODEL` - High-tier model name
- `{PROVIDER}_SONNET_MODEL` - Mid-tier model name
- `{PROVIDER}_HAIKU_MODEL` - Fast-tier model name

These are mapped to Claude Code's environment variables:
- `ANTHROPIC_AUTH_TOKEN`
- `ANTHROPIC_BASE_URL`
- `ANTHROPIC_DEFAULT_OPUS_MODEL`
- `ANTHROPIC_DEFAULT_SONNET_MODEL`
- `ANTHROPIC_DEFAULT_HAIKU_MODEL`

### Adding a New Provider
1. Add configuration variables to `start-claude.env.example`
2. Add validation in `install.sh` (required_vars array)
3. Add menu option in `start-claude.sh` (provider selection menu)
4. Add case handler in `start-claude.sh` (provider configuration section)

## Dependencies
- `jq` - JSON manipulation (auto-installed via Homebrew by install.sh)
- `bash` - Shell interpreter
- Claude CLI - Must be installed and path set in `CLAUDE_CLI` variable

---
> Source: [Liu-Lixin/claude-multi-provider-launcher](https://github.com/Liu-Lixin/claude-multi-provider-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
