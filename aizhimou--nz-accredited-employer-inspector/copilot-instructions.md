## nz-accredited-employer-inspector

> > **Notice for AI Agents**: This document is the master orientation and operational guide for working on `nz-accredited-employer-inspector`. Read this file first before inspecting or modifying code.

# AGENTS.md

> **Notice for AI Agents**: This document is the master orientation and operational guide for working on `nz-accredited-employer-inspector`. Read this file first before inspecting or modifying code.

---

## 1. Project Purpose

An open-source system enabling job seekers on LinkedIn and SEEK New Zealand to verify employer accreditation status with Immigration New Zealand (INZ) without leaving the job page.

- **Production Site**: https://nzaei.zemo.bio
- **Single Source of Truth (SSOT)**: [`docs/extension-api-ssot.md`](./docs/extension-api-ssot.md) — *All contract, state machine, and data model changes must align with this document.*

---

## 2. Inviolable System Invariants (Red Lines)

When modifying any part of this repository, you **MUST NOT** violate these design principles:

1. **No Worker-to-INZ Calls**: Cloudflare Workers **never** call INZ endpoints directly (avoids centralized IP blocks). Live INZ lookups are executed exclusively by the user's browser extension background script, which submits raw responses back to the Worker.
2. **Explicit User Action Only**: Page loads **never** trigger automatic live lookups. A live check occurs only when the user explicitly clicks *Check NZ accreditation*. A single click triggers **at most one** live INZ request.
3. **Strict Evidence & Trust Separation**:
   - **Official INZ Facts**: Legal name, trading name, 13-digit NZBN, accreditation expiry date.
   - **Community Association**: Mapping of platform page (LinkedIn/SEEK) to an NZBN, tracked via anonymized client hashes and confirmation counts.
   - **Derived Exact Match**: Ephemeral on-the-fly string equality between normalized platform display name and official/trading name. *Never persisted as a community confirmation.*
   - **No-Match Observation**: Negative cache inside a configured TTL (default 7 days). Proves only that a specific query yielded no published record at that time; not proof of non-accreditation.
4. **Server-Owned Provenance**: Clients cannot supply timestamps, verification status flags, or authority assertions. The Worker owns `last_verified_at` and evaluates expiry against Pacific/Auckland calendar dates.
5. **No Account Tracking**: The extension generates a random installation UUID (`crypto.randomUUID()`) sent as `X-Client-ID`. The Worker hashes it with SHA-256 before storing. Never extract or store LinkedIn/SEEK user account identities.

---

## 3. Monorepo Layout & Tech Stack

Each workspace is independently managed. Requirements: **Node.js >= 22**.

| Subproject | Path | Tech Stack | Purpose |
| :--- | :--- | :--- | :--- |
| **Extension** | [`extension/`](./extension) | WXT, Vite, TypeScript, Shadow DOM | Manifest V3 Chrome extension for LinkedIn & SEEK. |
| **API** | [`api/`](./api) | Cloudflare Workers, D1 (SQLite), R2 | Employer resolution, ingest, negative cache, FTS5 search, open-data crons. |
| **Landing** | [`landing/`](./landing) | Astro 5, Cloudflare Pages, Observable Plot | Product site, privacy policy, open-data catalog, LLM contexts (`llms.txt`). |
| **Refresh Script**| [`scripts/employer-refresh/`](./scripts/employer-refresh) | Node.js ESM | Operator-run batch refresh for near-expiry employers (outside extension path). |
| **Docs** | [`docs/`](./docs) | Markdown | Architecture SSOT and Cloudflare deployment runbooks. |

---

## 4. Architecture & Key Workflow

```text
[LinkedIn / SEEK Page]
       │ (User clicks "Check accreditation")
       ▼
[Extension Content Script] ──(postMessage)──► [Extension Background]
                                                      │
                       ┌──────────────────────────────┴──────────────────────────────┐
                       │ 1. POST /v1/employers/resolve                               │
                       ▼                                                             │
             [Cloudflare Worker]                                                     │
              (Checks D1 DB)                                                         │
                       │                                                             │
         ┌─────────────┴─────────────┐                                               │
         ▼                           ▼                                               │
   [Result Fresh]            [Lookup Required / Stale]                               │
         │                           │                                               │
         │ returns ASSOCIATED        │ returns INZ_LOOKUP_REQUIRED / REFRESH         │
         │ or NO_PUBLISHED_MATCH     ▼                                               │
         │                     [Extension Background]                                │
         │                               │                                           │
         │                               ├─ 2. POST to INZ official endpoint         │
         │                               ▼                                           │
         │                       [INZ Official API]                                  │
         │                               │                                           │
         │                 ┌─────────────┴─────────────┐                             │
         │                 ▼ (200 OK Results)          ▼ (400 "No Results")          │
         │          POST /ingest                 POST /no-match                      │
         │                 │                           │                             │
         │                 └─────────────┬─────────────┘                             │
         ▼                               ▼                                           │
    [Render Result / Candidate List in Shadow DOM Widget] ◄──────────────────────────┘
```

---

## 5. Development & Verification Guide

Always run checks in the specific subproject directory before submitting changes:

### Extension (`extension/`)
```bash
cd extension
npm install
npm run check        # Runs: typecheck (tsc) && test (vitest) && build (wxt)
npm run dev          # Local development with Chrome auto-reload
```

### Worker API (`api/`)
```bash
cd api
npm install
npm run check        # Runs: cf-typegen && typecheck && vitest run
npm run test         # Runs test suite
```
*Note: Database migrations live in `api/migrations/`. D1 schema changes must follow sequential numbering (e.g. `0010_...sql`).*

### Landing Site (`landing/`)
```bash
cd landing
npm install
npm run build        # Verifies static page generation & assets
npm run dev          # Local dev server
```

---

## 6. API Route Scope Summary

- **Extension API (`/v1/*`)**:
  - `POST /v1/employers/resolve`: Resolve platform identity to canonical employer or lookup state.
  - `POST /v1/employers/search`: User-initiated FTS5 keyword search across employer & trading names.
  - `POST /v1/employers/ingest`: Store raw valid positive INZ query responses.
  - `POST /v1/employers/no-match`: Record recognized INZ 400 "No Results" observations.
  - `POST /v1/employers/refresh`: Claim a per-NZBN rate-limited lease to refresh stale records.
  - `POST /v1/employers/associate`: Record/update a client's community confirmation for an entity.
  - *Header requirement*: `X-Client-ID` UUID is mandatory for all `/v1/employers/*` endpoints.

- **Public Read-Only API (`/public/v1/*`)**:
  - `GET /public/v1/employers/{nzbn}`: Read single employer by 13-digit NZBN.
  - `GET /public/v1/employers/search?q={query}`: Bounded keyword search.
  - *Constraint*: No authentication; strictly read-only; does not expose platform entities, client hashes, or refresh leases.

---
> Source: [aizhimou/nz-accredited-employer-inspector](https://github.com/aizhimou/nz-accredited-employer-inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
