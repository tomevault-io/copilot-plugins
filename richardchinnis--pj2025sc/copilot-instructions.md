## pj2025sc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PJ2025SC (Project 2025 Scorecard) is an evidence-based scoring system that tracks implementation status of ~1,500 policy proposals from "Project 2025: Mandate for Leadership" across 34 chapters. Built with Express + Nunjucks (server-side rendering), PostgreSQL, and a file-based inbox pipeline for AI agent workflows.

## Commands

```bash
npm start          # Express server + inbox watcher (default port 3000)
npm test           # Jest test suite (requires live PostgreSQL connection)
npm run migrate    # Bulk import from CH##/*_SCORECARD_DATA.json → PostgreSQL
npm run watcher    # Standalone inbox watcher (already included in npm start)
npx jest test/queries.test.js   # Run a single test file
```

## Architecture

**Server-side rendered master-detail UI** — no SPA framework. Left panel shows dashboard with 5 sections and 34 chapters; right panel loads chapter detail via AJAX (`?fragment=1`).

### Key Files

| File | Role |
|------|------|
| `server.js` | Express app, routes (`/`, `/chapter/:key`), in-memory cache with version-based invalidation |
| `db.js` | PostgreSQL pool (`DATABASE_URL` from `.env`) |
| `queries.js` | Data layer: `getOverall()`, `getSections()`, `getChapter(key)`, `getVersion()` |
| `watcher.js` | Monitors `inbox/{evidence,scores,verification}/` for JSON, processes → DB, bumps version, archives to `inbox/processed/` |
| `migrate.js` | One-time bulk import from `CH##/` JSON folders, drops and recreates all tables |
| `schema.sql` | 7 tables: chapters, authors, proposals, scores, evidence, verification, metadata |

### Data Flow

```
Agent research → JSON dropped in inbox/ → watcher.js validates & inserts → bumps metadata.version → cache invalidated → next request re-renders
```

### Database

- **Chapter keys**: `CH01`–`CH30`, with sub_ids for splits (`CH08a`, `CH08b`)
- **Proposal IDs**: `CH##-###` format (e.g., `CH01-001`)
- **Score scale**: -2 (fully blocked) to +2 (fully implemented), NULL for unscored
- **Evidence tiers**: 1 (official .gov) through 5 (social media)
- **Verification verdicts**: VERIFIED, PARTIAL, REDIRECT, PAYWALL, NOT FOUND, MISMATCH, HALLUCINATED, NOT CHECKED

### Caching

Single `metadata.version` integer in DB. Any watcher update bumps it, which invalidates the entire in-memory page cache on next request.

### Agent Workflows

Prompt templates in `prompts/` drive three agent operations:
- **full-rescore.md** — Re-score all proposals using existing evidence
- **research-zeros.md** — Find new evidence for zero-scored proposals
- **verify-urls.md** — Check evidence URLs and assign verification verdicts

Output JSON goes to the corresponding `inbox/` subdirectory for watcher processing.

## Testing

Tests connect to **live PostgreSQL** (no mocks). The test suite validates query functions and HTTP endpoints. Database must be populated via `npm run migrate` before tests will pass.

## Environment

Requires `.env` with:
```
DATABASE_URL=postgres://...
PORT=3000
```

---
> Source: [RichardChinnis/PJ2025SC](https://github.com/RichardChinnis/PJ2025SC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
