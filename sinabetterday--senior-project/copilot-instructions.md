## senior-project

> **FAIR** = Financial Accountability & Interest Review — a public web app for the **California FPPC** that cross-references **Form 700** financial disclosures with **city council agendas** to surface potential conflicts of interest.

# FAIR — workspace map (read this first)

**FAIR** = Financial Accountability & Interest Review — a public web app for the **California FPPC** that cross-references **Form 700** financial disclosures with **city council agendas** to surface potential conflicts of interest.

This repo uses a **three-layer** layout: (1) this file + `DIRECTORY.md`, (2) area `CONTEXT.md` files, (3) real code and data under `client/`, `server/`, etc.

## Layer 1 — Map (always relevant)

- **Public users:** browse/compare; **no** login.
- **Admins only:** Supabase Auth (single email/password; **no** public registration).
- **Raw Form 700:** archived in **Supabase Storage** (XLSX).
- **DB:** PostgreSQL on **Supabase**; ORM: **Prisma** (target stack).
- **Frontend:** React (Vite), Tailwind, React Router — **`client/`**.
- **Backend:** Node, Express — **`server/`** (hosting target: Railway).
- **Scraping:** **Legistar REST** where available; **Apify** for PDF-only cities.
- **Parsing:** `pdf-parse`, SheetJS `xlsx`.
- **Matching:** `fuse.js`; **Gemini 1.5 Flash** only for fuzzy scores **0.7–0.85** (entity resolution).
- **Email:** Resend (weekly admin digests).
- **Jobs:** **`node-cron`** nightly sync — **not** BullMQ/Redis.

## Layer 2 — Rooms (load by task)

| Area | Folder / doc |
|------|----------------|
| UI | `client/CONTEXT.md` |
| HTTP API & app server | `server/CONTEXT.md` |
| DB & auth & storage policy | `supabase/CONTEXT.md` |
| Agenda import scripts (Python) | `APIs/CONTEXT.md` |
| Form 700 ingestion & schedules | `docs/fair/form700-ingestion/CONTEXT.md` |
| Agendas (Legistar / Apify / PDF) | `docs/fair/agenda-ingestion/CONTEXT.md` |
| Fuse + Gemini resolution | `docs/fair/entity-resolution/CONTEXT.md` |
| Nightly sync | `docs/fair/cron-jobs/CONTEXT.md` |
| Resend digests | `docs/fair/admin-notifications/CONTEXT.md` |

## Layer 3 — Where work lands

- App code: `client/`, `server/`.
- SQL migrations: `supabase/migrations/`.
- Legacy/misc: `Backend/`, `workflows/`.

## Naming & routing

- Prefer **stable file names** so humans and tools can find artifacts (e.g. dated agendas, `draft` / `v2` Form 700 outputs).
- Before large edits, read **`DIRECTORY.md`** for task → files to read/skip.

## Form 700 schedule names (domain vocabulary)

Schedules **A, B, C, D, E, A-2** — see `docs/fair/form700-ingestion/CONTEXT.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SinaBetterDay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
