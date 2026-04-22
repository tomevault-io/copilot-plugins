## oh-my-bookmark

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Dev server:** `pnpm dev` (port 51731)
- **Build:** `pnpm build`
- **Type check:** `pnpm check`
- **CSV import:** `pnpm import` (runs `tsx scripts/import-csv.ts`)
- **D1 migrations:** `pnpm wrangler d1 migrations apply oh-my-bookmark-db --local` (local) / `--remote` (prod)
- **Deploy:** Automatic via GitHub Actions on push to `main`; or manual `pnpm wrangler deploy`

## Tech Stack

- **SvelteKit 2** with **Svelte 5** (runes: `$state`, `$props`, `$derived`) on **Vite 7**
- **Cloudflare Workers** runtime with **D1** (SQLite) database
- **Tailwind CSS 4** with **bits-ui** for headless UI components
- **pnpm** package manager (v10)
- No test framework configured

## Architecture

This is a bookmark management app deployed to Cloudflare Workers.

**Database layer** (`src/lib/server/db.ts`): All D1 queries are centralized here. Functions accept a `D1Database` parameter obtained from `platform.env.DB`. Folder hierarchy is built client-side from a flat list via `buildFolderTree()`.

**API routes** (`src/routes/api/`): REST endpoints for bookmarks, folders, tags, and CSV import. Standard GET/POST/PUT/DELETE with JSON bodies.

**Pages** (`src/routes/`): File-based routing with server load functions (`+page.server.ts`, `+layout.server.ts`). After mutations, pages call `invalidateAll()` to refresh data.

**Components** (`src/lib/components/`): Svelte 5 components using runes. `BookmarkList` handles display + inline edit/delete; `FolderTree` is recursive; `BookmarkForm` handles create/edit.

**Types** (`src/lib/types.ts`): Shared interfaces — `Bookmark`, `BookmarkWithTags`, `Folder`, `Tag`, `PaginatedResult<T>`.

## Database

Single D1 database bound as `DB`. Schema in `migrations/0001_init.sql` with four tables: `folders`, `bookmarks`, `tags`, `bookmark_tags` (junction). Tags use many-to-many via `syncBookmarkTags()`.

## Key Patterns

- Platform env access: `platform.env.DB` in server load functions and API routes
- Tag sync: `syncBookmarkTags(db, bookmarkId, tagNames[])` handles create-or-get + junction table
- Folder paths: stored as `/parent/child` strings with unique constraint; `buildFolderTree()` reconstructs hierarchy
- Pagination: `PaginatedResult<T>` with configurable page sizes (10/20/50/100)
- No authentication — all endpoints are public

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darknight) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
