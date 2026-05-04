## tampermonkey-crawler

> Conventions and pointers for working on this codebase.

# Agent Guide

Conventions and pointers for working on this codebase.

## Project structure

Two npm workspaces (`server/` and `tampermonkey/`) in a monorepo. They share no code -- communication is over HTTP on port 4242. See [docs/architecture.md](docs/architecture.md) for the full data flow.

## Key conventions

- **Conventional commits**: `feat(scope):`, `fix(scope):`, `refactor:`, `chore:`, `docs:`
- **Crawler-as-directory**: Each crawler lives in `tampermonkey/src/crawlers/<site>/` with `extractors.ts` (pure DOM parsing, testable with jsdom) and `index.ts` (orchestration that wires extractors to network I/O).
- **Raw storage only**: The server stores raw crawl data as JSON in `raw_crawls`. It does NOT transform data into any output format. Transformation is the responsibility of external consumers.
- **Content-addressed blobs**: Media (images, video) is stored by content hash for deduplication. The `blobs` table stores binary data; `raw_crawls` reference blob hashes.
- **Tests**: vitest for both workspaces. Tampermonkey tests use jsdom and HTML fixtures in `__fixtures__/` dirs. Server tests are pure node. Run with `make test`.
- **No shared types**: The server and tampermonkey each define their own types. The HTTP payload format is the implicit contract.

## Where to find things

| Topic | File |
|---|---|
| Data flow and DB schema | [docs/architecture.md](docs/architecture.md) |
| API endpoints | [docs/architecture.md](docs/architecture.md#api-endpoints) |
| Server routes and storage | [docs/server.md](docs/server.md) |
| Crawlers and extractors | [docs/tampermonkey.md](docs/tampermonkey.md) |
| Adding a new site | [docs/troubleshooting.md](docs/troubleshooting.md#6-adding-a-new-site) |
| Selector breakage and fixtures | [docs/troubleshooting.md](docs/troubleshooting.md#1-selector-breakage-most-common-issue) |
| Dashboard UI | `server/public/index.html` (single-file, Tailwind via CDN) |
| Userscript build config | `tampermonkey/vite.config.ts` (`@connect`, `@grant`, `@match`) |
| SQLite schema | `server/src/db.ts` |
| Environment variables | `.env` at project root (`PORT`) |

## Make targets

| Command | What it does |
|---|---|
| `make dev` | Start the server (Fastify on port 4242) |
| `make build` | Build both server and tampermonkey userscript |
| `make test` | Run all tests in both workspaces |
| `make ci` | Format check + lint + build + test (runs on pre-commit) |

After changing crawler code: `make build && make dev`, then reinstall the userscript at `http://localhost:4242/tampermonkey.user.js`.

## Common tasks

**Adding a new site crawler**: Follow the step-by-step in [docs/troubleshooting.md, section 6](docs/troubleshooting.md#6-adding-a-new-site). Touches both workspaces.

**Fixing broken selectors**: Update the extractor in `tampermonkey/src/crawlers/<site>/extractors.ts`, capture a new HTML fixture, update tests, rebuild. See [docs/troubleshooting.md, section 1](docs/troubleshooting.md#1-selector-breakage-most-common-issue).

**Changing the DB schema**: Edit `server/src/db.ts`. Tables are created with `CREATE TABLE IF NOT EXISTS`, so new columns require a migration or deleting `crawler.db`.

---
> Source: [adlio/tampermonkey-crawler](https://github.com/adlio/tampermonkey-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
