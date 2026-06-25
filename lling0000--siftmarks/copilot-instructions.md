## siftmarks

> SiftMarks is a local-first TypeScript monorepo for turning browser bookmarks into searchable AI memory.

# SiftMarks Agent Notes

## Project Shape

SiftMarks is a local-first TypeScript monorepo for turning browser bookmarks into searchable AI memory.

- `packages/shared`: shared types and utility functions.
- `packages/db`: SQLite schema and data access through `better-sqlite3`.
- `packages/core`: bookmark import, parsing, search, duplicate detection, rescue suggestions, and browser bookmark detection.
- `packages/ai`: mock, OpenAI-compatible, and Ollama-compatible AI providers.
- `packages/indexer`: FTS indexing, summaries, tags, and embeddings.
- `apps/cli`: `siftmarks` command-line interface.
- `apps/mcp-server`: MCP stdio server for AI clients.
- `apps/web`: Next.js local dashboard and API routes.
- `apps/chrome-extension`: Chrome extension that talks to the local web API on port `4399`.

## Commands

- Install: `npm install`
- Build non-web packages, CLI, and MCP server: `npm run build:packages`
- Build everything: `npm run build`
- Run the web dashboard: `npm run dev`
- Run the CLI after building: `node apps/cli/dist/index.js`
- Typecheck: `npm run typecheck`

Use `SIFTMARKS_HOME=/tmp/siftmarks-test` for smoke tests that should not touch the user's real `~/.siftmarks` database.

## Current Caveats

- The web app has its own rules in `apps/web/AGENTS.md`; read that file before editing anything under `apps/web`.
- The root `typecheck` command can surface web/Next configuration issues. Do not assume package-level builds prove the web app is production-buildable.
- The Chrome extension assumes the web app is running at `http://localhost:4399`.
- AI calls must stay disabled unless a provider is explicitly configured; mock mode is the default.

## Working Rules

- Do not perform git actions unless the user explicitly asks.
- Keep changes scoped to the requested area.
- Prefer existing package boundaries over adding new cross-package shortcuts.
- Treat local bookmark databases and browser profile files as user data; use temporary data directories for tests when possible.
- When changing data writes, preserve the local-first behavior and avoid silent external network calls.

---
> Source: [Lling0000/SiftMarks](https://github.com/Lling0000/SiftMarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
