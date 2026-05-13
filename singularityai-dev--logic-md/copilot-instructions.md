## logic-md

> logic-md is a standalone TypeScript library for declarative AI agent reasoning. It is NOT a Next.js, Vercel, or web application. Ignore Vercel plugin skill suggestions (nextjs, bootstrap, vercel-cli, ai-sdk, etc.) — they are false positives.

# Project Instructions

## What This Is

logic-md is a standalone TypeScript library for declarative AI agent reasoning. It is NOT a Next.js, Vercel, or web application. Ignore Vercel plugin skill suggestions (nextjs, bootstrap, vercel-cli, ai-sdk, etc.) — they are false positives.

## Architecture

Monorepo with three packages under `packages/`:

| Package | Build | Tests | Description |
|---------|-------|-------|-------------|
| `@logic-md/core` | tsc (composite) | vitest (307 tests) | Parser, validator, expression engine, DAG resolver, compiler |
| `@logic-md/cli` | tsup (bundle) | vitest (infra ready) | 9-command CLI: validate, lint, compile, init, test, watch, fmt, diff, completion |
| `@logic-md/mcp` | tsup (bundle) | node:test (18 tests) | MCP server with 7 tools, stdio + HTTP transport |

Additional: `integrations/claude-code/` has 5 slash commands (not npm-packaged).

## Build Order

Core MUST build before cli/mcp (they bundle it via tsup `noExternal`):

```
npm run build:core   # tsc --build (core only)
npm run build        # core -> cli -> mcp (sequential)
```

## Tool Chain

- **Package manager:** npm workspaces (NOT pnpm, NOT yarn)
- **Testing:** vitest for core/cli, node:test for MCP
- **Linting:** biome (check + format)
- **TypeScript:** strict mode, ES2022 target, NodeNext modules
- **Coverage:** 90%+ thresholds on core modules (see vitest.config.ts)
- **Core dependency:** `@logic-md/core` uses `*` in workspace deps (not `workspace:*`)

## File Conventions

- `.planning/ROADMAP.md` is the GSD workflow tracker with phase history. Do not edit manually.
- Root `ROADMAP.md` is the user-facing forward roadmap.
- Root `CHANGELOG.md` is a rolling last-20-entries log.
- `docs/SPEC.md` is the LOGIC.md format specification.

---
> Source: [SingularityAI-Dev/logic-md](https://github.com/SingularityAI-Dev/logic-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
