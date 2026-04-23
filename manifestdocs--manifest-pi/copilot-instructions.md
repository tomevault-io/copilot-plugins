## manifest-pi

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Overview

`@manifest/pi` is a Pi coding agent package that integrates Manifest feature management tools. It calls the Manifest HTTP API directly via `fetch()` — no MCP protocol layer.

## Build & Test

```bash
pnpm install                    # Install dependencies
pnpm build                      # Compile TypeScript (src/ -> dist/)
pnpm test                       # Run tests in watch mode
pnpm test:run                   # Run tests once (CI)
pnpm check                      # Type-check without emitting
```

## Architecture

- **`src/types.ts`** — Domain types (ported from manifest-svelte, zero dependencies)
- **`src/client.ts`** — HTTP client wrapping fetch() with typed methods
- **`src/format.ts`** — Text rendering (trees, tables, state symbols, time buckets)
- **`src/tools/*.ts`** — Tool definitions using TypeBox schemas, registered via `pi.registerTool()`
- **`extensions/manifest.ts`** — Entry point: creates client, registers all tools
- **`skills/*.md`** — Workflow scripts (ported from manifest-plugin)
- **`prompts/manifest.md`** — System instructions for agents
- **`agents/product-engineer.md`** — Product engineer agent definition

## Conventions

- **TypeBox schemas** for all tool parameters — use `StringEnum` from `@mariozechner/pi-ai` for string enums
- **Peer dependencies** on Pi runtime (`@mariozechner/pi-coding-agent`, `@sinclair/typebox`) — never bundled
- **`agent_type: "pi"`** for feature claims (server stores as free-form TEXT)
- **TDD**: Write tests first, then implement
- **Vitest** with BDD-style `describe`/`it` blocks
- **Mock fetch** at the global level for client/tool tests

## Tool Naming

All tools prefixed with `manifest_`: `manifest_list_projects`, `manifest_start_feature`, etc.

## Coding Guidelines

See `coding-guidelines.md` for TypeScript conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manifestdocs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
