## agent-docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Build & Dev Commands

```bash
pnpm run build          # Build with tsup (ESM, Node 18 target)
pnpm run dev            # Build in watch mode
pnpm run lint           # Type-check (tsc --noEmit)
pnpm run test           # Run all tests
node --import tsx --test test/resolvers/mcp.test.ts  # Run a single test file
```

Tests use Node's built-in `node:test` runner with `tsx` for TypeScript — no test framework dependency.

## Architecture

**CLI tool + MCP server** that scans `package.json` dependencies, discovers documentation sources for each library, and generates `.agent-docs/index.md` with pointers to those sources. The index is referenced from agent instruction files (CLAUDE.md, AGENTS.md, etc.) so AI agents always know where to find docs.

### Three entry points

- `bin/agent-docs.ts` → CLI entry, delegates to `src/cli.ts` which routes to command handlers
- `src/mcp.ts` → MCP server (stdio transport) exposing `init`, `update`, `add_resource`, `get_index` tools
- `src/index.ts` → Library exports for programmatic use

### Resolution pipeline (`src/commands/init.ts`)

For each dependency, resolution happens in two phases:
1. **npm registry** (`src/resolvers/registry.ts`) — fetches homepage/repo URL (others depend on this)
2. **Parallel resolvers** — `llms-txt`, `context7`, `mcp` each return a `DocSource | null`
3. **Fallback** — if no resolver found anything, use homepage as docs URL

Concurrency is controlled by `src/utils/concurrency.ts` (promise-based limiter, default 10).

### Key modules

- `src/resolvers/mcp.ts` — looks up packages in `data/known-mcps.json` (curated registry of MCP endpoints)
- `src/resolvers/llms-txt.ts` — HEAD requests to `<homepage>/llms.txt`
- `src/resolvers/context7.ts` — queries Context7 API with rate limiting
- `src/parsers/detect-agent-file.ts` — finds the right agent instruction file to inject into (follows `@AGENTS.md` references in CLAUDE.md)
- `src/generators/injection.ts` — idempotent injection between `<!-- agent-docs:start -->` / `<!-- agent-docs:end -->` markers
- `src/cache/file-cache.ts` — file-based cache in `.agent-docs/.cache/` with TTL support
- `src/config/config-manager.ts` — loads/saves `.agent-docs/config.json`

### Type system

Core types in `src/resolvers/types.ts`: `DocSource` (with `type` discriminator: `llms-txt | context7 | mcp | docs-url | skill`), `ResolvedDependency`, `ManualResource`, `Config`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erikpr1994) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
