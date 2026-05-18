## kb-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development

```bash
pnpm build          # rolldown → dist/main.js (ESM bundle)
pnpm test           # vitest run (all tests)
pnpm dev            # build + run CLI
pnpm clean          # rm -rf dist
```

Run a single test file: `npx vitest run tests/remote-config.test.ts`

The CLI entry point is `bin.js` → `dist/main.js`. After building, test locally with `node ./dist/main.js <command>`.

## Architecture

**Framework**: [moostjs](https://moost.org) — decorator-driven CLI (`@moostjs/event-cli`) and HTTP (`@moostjs/event-http`) framework. Controllers use `@Cli()`, `@Param()`, `@CliOption()`, `@Description()` decorators.

**Build**: Rolldown bundles `src/main.ts` into a single ESM file. The `unplugin-atscript` plugin compiles `.as` model files. All heavy dependencies are externalized (not bundled).

**Data storage**: `~/.kb/` directory. Each wiki is `~/.kb/{name}/docs/` (markdown files) + `index.db` (SQLite for metadata, FTS5, vectors, links). Markdown files are source of truth; SQLite is a rebuildable index.

### Key layers

```
Controllers (CLI commands)
    ↓ resolveWiki() → WikiRef (local or remote)
WikiGateway → getOps(ref) → WikiOps interface
    ├── LocalWikiOps → existing services (storage, search, index, workflow)
    └── RemoteWikiOps → RemoteClient → HTTP to remote kb server
```

**WikiRef routing**: `ConfigService.resolveWiki()` returns a discriminated union (`{ type: 'local' | 'remote', ... }`). The gateway creates the appropriate ops implementation. Controllers don't know or care whether a wiki is local or remote.

**ApiController is NOT routed through the gateway** — it always operates locally because it IS the remote endpoint that RemoteClient calls.

### Services (src/services/)

- `container.ts` — singleton DI container, instantiates all services
- `config.service.ts` — global config + CWD config walking + wiki resolution
- `remote-config.service.ts` — manages `~/.kb/remotes.json` (cached in memory)
- `storage.service.ts` — markdown file read/write (sync, filesystem)
- `index.service.ts` — atscript-db with SQLite (documents, links, chunks tables)
- `fts.service.ts` — raw SQL FTS5 table (not contentless; BM25 with weights)
- `vector.service.ts` — raw sqlite-vec for embeddings (delete+insert pattern, not REPLACE)
- `embedding.service.ts` — lazy HuggingFace all-MiniLM-L6-v2 (384-dim, cached in ~/.kb/.models/)
- `search.service.ts` — hybrid search via Reciprocal Rank Fusion (RRF)
- `doc-workflow.service.ts` — orchestrates index+FTS+vector operations
- `wiki-ops.ts` — WikiOps interface + LocalWikiOps + RemoteWikiOps
- `wiki-gateway.service.ts` — routes WikiRef to correct ops implementation
- `remote-client.ts` — HTTP client (native fetch) for remote API calls

### Models (src/models/)

AtScript `.as` files compiled by `unplugin-atscript` at build time. Type declarations are generated alongside as `<name>.as.d.ts` by `asc -f dts` (or the VSCode extension's on-save hook). TS picks these up correctly under `moduleResolution: "Bundler"` (set in [tsconfig.json](tsconfig.json)) — `NodeNext` would strip the `.as` extension and miss them.

### Content (content/)

Bundled markdown files served via `kb skill [workflow]`. These are agent-facing instructions (no API docs — CLI only).

## Conventions

- All services take `kb: string` (wiki name) as first param for wiki-scoped operations
- `toFilename(id)` auto-appends `.md`; `toDocId(filename)` strips it
- Links use standard markdown relative format: `[text](./filename.md)`
- Categories are free-form strings in frontmatter, not directories (flat file structure)
- Config resolution: explicit `--wiki` flag → CWD `kb.config.json` → global default
- Remote wiki names must not conflict with local wiki names (error until aliased)

## Testing

Tests live in `tests/`. Use vitest with ESM. For mocking `os.homedir()` (needed to test config services), use `vi.mock('node:os')` with `vi.importActual`.

Global fetch is mocked via `vi.stubGlobal('fetch', mockFn)` for RemoteClient tests.

---
> Source: [mav-rik/kb-cli](https://github.com/mav-rik/kb-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
