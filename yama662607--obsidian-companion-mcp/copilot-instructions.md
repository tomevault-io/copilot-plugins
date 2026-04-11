## obsidian-companion-mcp

> This document provides guidelines for Gemini agents working on this project.

# Gemini Agent Guidelines

This document provides guidelines for Gemini agents working on this project.

## Project Overview

Obsidian Companion MCP - A hybrid Model Context Protocol ecosystem for Obsidian.

- **MCP (`/mcp`)**: Node.js MCP server that proxies AI agent requests to the Obsidian plugin
- **Plugin (`/plugin`)**: Obsidian plugin for semantic indexing and real-time editor context

## Justfile Usage

This project uses `just` for task automation. Follow these commands:

### Required Commands
- `just check` — Run all read-only checks (format, lint, typecheck)
- `just fix` — Apply auto-fixable formatting and lint issues

### Workflow
1. After editing: Run `just check`
2. If errors: Run `just fix`, then `just check` again
3. Only commit when `just check` passes

### All Commands
- `just setup` — Install dependencies for all subprojects
- `just check` — Run all quality checks (CI gate)
- `just fix` — Apply auto-fixes
- `just test [args]` — Run tests
- `just build` — Build all subprojects
- `just clean` — Remove build artifacts
- `just dev-mcp` — Start mcp development
- `just dev-plugin` — Start plugin development
- `just mcp-build` — Build mcp only
- `just plugin-build` — Build plugin only
- `just upgrade` — Upgrade dependencies

## Development Notes

- TypeScript project with monorepo structure
- Each subproject has its own `package.json`
- No package manager lockfile at root level (each subproject manages its own)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yama662607)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yama662607)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
