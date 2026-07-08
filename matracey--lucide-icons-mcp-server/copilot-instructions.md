## lucide-icons-mcp-server

> npm run build          # TypeScript compilation (tsc)

# Copilot Instructions for lucide-icons-mcp-server

## Build, Test & Lint

```bash
npm run build          # TypeScript compilation (tsc)
npm test               # Run all tests (vitest)
npm run test:watch     # Watch mode
npm run test:coverage  # With coverage report
npm run lint           # ESLint on src/ and scripts/
npm run format:check   # Prettier check
npm run format         # Prettier auto-fix
npm run validate       # Run all CI checks (format, lint, build, test:coverage)
```

**Important:** Always run `npm run validate` after making changes to ensure nothing is broken. This runs the same checks as CI.

Run a single test file:

```bash
npx vitest run src/__tests__/server.test.ts
```

Run a single test by name:

```bash
npx vitest run -t "should return results for a valid query"
```

## Architecture

This is an MCP (Model Context Protocol) server that exposes Lucide icon documentation as tools for AI assistants. It communicates over **stdio transport**.

### Data Flow

```
generate-knowledge-base.ts → icons.json → search/formatters → MCP tools (stdio)
```

1. **Knowledge base generation** (`npm run generate`): The `scripts/generate-knowledge-base.ts` script fetches icon metadata from the Lucide APIs (`/api/categories`, `/api/tags`) and SVG content from the `lucide-icons/lucide` GitHub repo, then outputs `src/data/generated/icons.json`. This is a dev-time step — the generated JSON is committed to the repo.

2. **Runtime**: `src/index.ts` creates the MCP server and connects it via `StdioServerTransport`. The server (defined in `src/server.ts`) registers 6 tools that query the static knowledge base through `src/search.ts` and format results via `src/formatters.ts`.

### Key Source Files

- `src/server.ts` — Registers all MCP tools with Zod schemas. This is where tool definitions and handlers live.
- `src/search.ts` — Token-based search engine with weighted scoring (exact name: +100, name contains: +50, token match: +30, etc.).
- `src/formatters.ts` — Converts icons and search results into markdown strings for tool responses. Generates per-package code examples from templates.
- `src/types.ts` — Core type definitions (`LucideIcon`, `LucidePackage`, `CodeExample`, `SearchResult`).
- `src/constants.ts` — Package definitions (all 11 Lucide packages), categories, base URLs.
- `src/data/generated/` — Auto-generated from `npm run generate`. Do not edit manually.

### Adding a New Tool

Edit `src/server.ts` and register a new tool using `server.tool()`. See the existing `lucide_search` tool for the pattern.

## Conventions

- **Formatting**: Prettier with no semicolons, single quotes, trailing commas (es5), 100 char print width.
- **Commit messages**: Follow [conventional commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `chore:`, `docs:`, `test:`).
- **Test coverage**: Maintain above 90%. Tests live in `src/__tests__/` mirroring source file names.
- **Integration tests**: Use `InMemoryTransport.createLinkedPair()` from the MCP SDK to test server tools end-to-end without stdio.
- **ESM**: The project uses ES modules (`"type": "module"` in package.json). Use `.js` extensions in TypeScript import paths.
- **Tool responses**: All MCP tool handlers return `{ content: [{ type: 'text' as const, text: string }] }`. Error responses add `isError: true`.

---
> Source: [matracey/lucide-icons-mcp-server](https://github.com/matracey/lucide-icons-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
