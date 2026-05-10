## danish-standards-mcp

> MCP server for Danish government cybersecurity standards.

# CLAUDE.md - Danish Standards MCP

MCP server for Danish government cybersecurity standards.

## Build & Test

```bash
npm install
npm run build         # TypeScript compile
npm run lint          # Type check only
npm test              # Unit tests
npm run test:contract # Golden contract tests
```

## Architecture

- `src/tools/*.ts` -- one file per tool, exports handler function
- `src/db.ts` -- SQLite connection singleton (WASM via @ansvar/mcp-sqlite)
- `src/response-meta.ts` -- shared _meta helper
- `src/index.ts` -- stdio entry point
- `src/http-server.ts` -- HTTP entry point with /health
- `data/standards.db` -- bundled SQLite (gitignored, built by scripts/build-db.ts)
- `scripts/seed-test-db.ts` -- creates minimal test DB

## Anti-Slop

Do not use: delve, beacon, tapestry, realm, testament, embark, endeavor, paramount, unparalleled, bespoke, multifaceted, holistic, cutting-edge, groundbreaking, game-changer, transformative, revolutionary, pioneering, trailblazing, harness, foster, underscore, leverage, utilize, unleash, seamless, facilitate.

---
> Source: [Ansvar-Systems/danish-standards-mcp](https://github.com/Ansvar-Systems/danish-standards-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
