## agent-kit

> - `mise run format` — Format all code (ruff for Python, dprint for TS/JSON/YAML/MD)

# Agent Instructions

## Build/Lint/Test Commands

- `mise run format` — Format all code (ruff for Python, dprint for TS/JSON/YAML/MD)

## Architecture

- `plugins/` — Claude Code plugins
- `extensions/` — pi extensions
- `mcps/` — MCP servers
- `skills/` — Agent skills

## Marketplace

- When adding new plugins or skills, update `.claude-plugin/marketplace.json` to register them

## Code Style

- **Python:** Python 3.12+, ruff (88-char lines, double quotes), snake_case, type hints required
- **TypeScript:** Strict mode, camelCase functions, PascalCase types, Zod for validation, biome for linting
- **Go:** Standard gofmt, internal/ for private packages
- **Commits:** Emoji Conventional Commits (e.g., `✨ feat:`, `🐛 fix:`, `♻️ refactor:`)
- Never commit secrets; use env vars for credentials

---
> Source: [vaayne/agent-kit](https://github.com/vaayne/agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
