## bangumi-downloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The project uses **bun** as the package manager and runtime (locally and in production). Lockfile is `bun.lock`.

- `bun install` — install deps (use `--frozen-lockfile` in CI/deploy)
- `bun run dev` — start Next.js dev server at http://localhost:3000
- `bun run build` / `bun run start` — production build / serve
- `bun run lint` — ESLint (flat config in `eslint.config.mjs`, extends `next/core-web-vitals` + `next/typescript`)
- `bun run typecheck` — `tsc --noEmit`
- `bun run test` — Vitest

## Environment

Copy `.env.example` → `.env.local`. Relevant vars, all read server-side:
- `BANGUMI_MOE_API_BASE`, `BANGUMI_MOE_SEARCH_PATH` — bangumi.moe upstream (path is configurable because the real endpoint varies)
- `DHT_INDEXER_URL`, `DHT_INDEXER_AUTH_TOKEN` — external DHT index service. The app does **not** crawl DHT itself; it expects a separate worker service to exist at that URL.

## Architecture

Next.js App Router app that aggregates torrent search results from multiple upstream sources behind a single API.

- `app/page.tsx` — client search UI, calls `/api/search?q=...&sources=...`
- `app/api/search/route.ts` — thin HTTP adapter; parses query params, validates `sources`, delegates to `searchTorrents`
- `lib/search.ts` — aggregator: fans out to all enabled providers in parallel (`Promise.all`), merges items, sorts by `publishedAt` desc, collects per-provider warnings
- `lib/providers/*` — one file per upstream. Each implements `SearchProvider { source, search(query) }` from `lib/types.ts` and returns a `ProviderResult` with normalized `SearchResultItem`s. Provider failures should surface as `warnings`, not thrown errors, so one bad source doesn't kill the aggregate response.
- `lib/types.ts` — the canonical shape. `SearchSource` is the discriminant ("bangumi-moe" | "dht"); adding a new source means extending this union, adding a provider class, and registering it in the `providers` array in `lib/search.ts`.

Path alias `@/*` → repo root (see `tsconfig.json`).

Key invariants to preserve when editing:
- All upstream tokens/URLs stay server-side — never import provider modules from client components.
- Provider output must conform to `SearchResultItem` (unified title/time/size/magnet/torrent/tags/seeders fields) so the UI and sort in `lib/search.ts` keep working.

---
> Source: [madeye/bangumi-downloader](https://github.com/madeye/bangumi-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
