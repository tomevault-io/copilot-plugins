## repo-context-mcp

> MCP + CLI tool that helps coding agents understand repositories via:

# AGENTS.md — repo-context-mcp

## Project

MCP + CLI tool that helps coding agents understand repositories via:

- `repo_map` — tree + entrypoints
- `search_code` — substring search
- `pack_context` — token-aware markdown pack

## Layout

- `src/server.ts` — MCP stdio server
- `src/cli.ts` — dual CLI entry (`map` / `search` / `pack` / `serve`)
- `src/walk.ts` / `gitignore.ts` / `repo-map.ts` / `search.ts` / `pack.ts` — pure core
- `tests/` — Vitest
- `examples/sample-repo/` — demo tree
- `skills/repo-context/` — agent skill doc

## Commands

```bash
npm install
npm test
npm run lint
npm run build
node dist/cli.js map examples/sample-repo
node dist/cli.js search login examples/sample-repo
node dist/cli.js pack examples/sample-repo --focus auth
```

## Conventions

- TypeScript strict, ESM (`NodeNext`)
- Do not write diagnostics to **stdout** in MCP mode (stdio protocol)
- Prefer pure functions + unit tests over integration-only coverage
- Keep dependencies minimal

## Security

- All file reads must go through `safeResolve` / `readTextFile`
- Never follow paths outside the configured root

---
> Source: [nduc99911/repo-context-mcp](https://github.com/nduc99911/repo-context-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
