## richard-med

> **The single source of truth for this repo is [`CLAUDE.md`](./CLAUDE.md). Read it

# Agent Instructions — Richard Med

**The single source of truth for this repo is [`CLAUDE.md`](./CLAUDE.md). Read it
fully before doing anything.** It defines the goal, the winning thesis, the agreed
2-person scope, architecture, data model, normalization plan, data sources, API
surface, UX rules, build order, and engineering conventions.

This file exists so non-Claude agents (Codex, Cursor, etc.) don't miss it.
`CLAUDE.md` and `AGENTS.md` must stay in agreement — if you update the plan, update
`CLAUDE.md` (canonical) and keep this pointer accurate.

## TL;DR (full detail in CLAUDE.md)

- **Goal:** "Aviasales for medical services in Kazakhstan" — search one service,
  compare real clinic prices with source proof, freshness, map, and route.
- **Thesis:** a _trusted price intelligence product_, not a scraper + table.
- **Hard rules:** no live scraping in the user path (search hits the DB only);
  one broken source never breaks the run; never overwrite raw evidence; never show
  data >30 days old as current; public data only (robots.txt, rate limits, no PII,
  2GIS not scraped).
- **Stack:** FastAPI + SQLAlchemy + PostgreSQL (`backend/`), Vite + React + TS +
  Tailwind (`frontend/`).
- **Conventions:** immutable data, small files (<800 lines), minimal comments,
  explicit error handling, TDD with "should" test names, Conventional Commits,
  **never push without explicit permission.**
- **First task:** import the `Services_Clean` catalog (1,281 services) and seed
  `service_aliases`, then build KDL + DOQ adapters.

---
> Source: [SuWh1/richard-med](https://github.com/SuWh1/richard-med) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
