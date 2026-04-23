## lumen

> - The repo root is the shared agent integration surface. Keep `.claude-plugin/`, `.codex/`, `.cursor-plugin/`, `.cursor/`, `.opencode/`, `hooks/`, `skills/`, `scripts/`, and `package.json` aligned where they represent the same Lumen distribution.

# AGENTS.md

## Repository Expectations

- The repo root is the shared agent integration surface. Keep `.claude-plugin/`, `.codex/`, `.cursor-plugin/`, `.cursor/`, `.opencode/`, `hooks/`, `skills/`, `scripts/`, and `package.json` aligned where they represent the same Lumen distribution.
- Do not add a repo-root `mcp.json` or repo-root `.codex-plugin/`. Claude Code reads repo-root `mcp.json` as project-scoped MCP config, which would change behavior for this repository. The Cursor MCP config lives in `.cursor/mcp.json`.
- When changing plugin metadata for releases, keep `.claude-plugin/plugin.json`, `.cursor-plugin/plugin.json`, and `package.json` aligned unless a difference is intentionally product-specific.

## Commands

- `make build-local` builds the local binary.
- `make test` runs the Go test suite.
- `make lint` runs `golangci-lint`.

---
> Source: [ory/lumen](https://github.com/ory/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
