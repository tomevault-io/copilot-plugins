## grc-intra-company-acct-coding

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (Next.js)
npm run build    # Production build
npm run lint     # Run ESLint
npm start        # Start production server
```

There are no automated tests in this codebase.

Docker build uses a multi-stage Node 20 image; the app runs on port 3000.

## Environment

Copy `.env.local` with these variables to connect to the SQL Server database:

```
SQL_SERVER_HOST
SQL_SERVER_PORT          # default 1433
SQL_SERVER_DATABASE
SQL_SERVER_USER
SQL_SERVER_PASSWORD
SQL_SERVER_INSTANCE      # optional
SQL_SERVER_CONNECTION_TIMEOUT
SQL_SERVER_REQUEST_TIMEOUT
```

## Architecture

**Next.js 15 App Router** full-stack app. Client components use `"use client"`. API routes in `src/app/api/` run server-side with direct SQL Server access via `mssql`.

**Data flow:**
1. `GET /api/tickets` → `src/lib/sql-server.ts` executes a ~540-line CTE query joining multiple ticket source tables (`Tkbatch`, `Tkhist1`, `Tkeother`, `Tkohist`) with job/customer/PE/PM lookup tables
2. Results render in `TicketsGrid.tsx` (the central component, ~2300 lines)
3. Account code edits are validated in real-time via `POST /api/account/validate` → stored procedure `dbo.GRC_Intra_ValidateAccountCodes`
4. `POST /api/tickets/save` upserts validated codes via `dbo.GRC_Upsert_Intra_Ticket_AccountCode_Working_To_TEST`

**Key source files:**
- `src/lib/sql-server.ts` — all database logic: connection pool, the main ticket query, save/validate operations
- `src/components/grids/TicketsGrid.tsx` — entire grid UI: filtering, sorting, inline editing, drag-to-fill, validation state, save flow
- `src/components/grids/HeaderFilter.tsx` — column-level filter dropdowns rendered via React portals
- `src/components/grids/DraggableCell.tsx` — HTML5 drag-and-drop for copying account codes across rows
- `src/types/grids.ts` — shared TypeScript types for ticket rows, columns, filter state

**State management:** All state lives in `TicketsGrid.tsx` via React hooks. Column order preferences are persisted to `localStorage`. No external state library is used.

**Filtering system:** Two tiers — top-level filters (Job, Customer, Order, PE, PM, Source, date range) and per-column header filters. Filter options are computed with `useMemo` and are dependent on each other (selecting a job narrows customer options, etc.).

**Validation lifecycle:** Each row tracks a `validationStatus` field (`'pending' | 'validating' | 'valid' | 'invalid'`). Editing an account cell triggers a debounced API call. The save button is blocked until all edited rows are valid.

## Tech Stack

- React 19 + Next.js 15 (App Router)
- TypeScript 5
- Tailwind CSS v4
- `react-data-grid` v7.0.0-beta.55 (beta — API may differ from stable docs)
- `mssql` v12 for SQL Server connectivity
- Path alias: `@/*` → `./src/*`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/graniterock)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/graniterock)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
