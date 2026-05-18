## og-dealer-app

> Before writing or modifying ANY Supabase query:

# OGDealer Project Rules

## Database Query Rules (MANDATORY)

Before writing or modifying ANY Supabase query:

1. **Read `DATABASE_SCHEMA.md`** - Contains the complete live schema (78 tables) with all column names, types, and defaults. Generated from the Supabase OpenAPI endpoint.
2. **Read `QUERY_RULES.md`** - Contains mandatory rules for Supabase queries including column validation, FK join requirements, and `.order()` syntax.
3. **Check `CONNECTION_AUDIT.md`** - Contains every Supabase query in the codebase with file, line number, table, operation, and validation status.

### Key Rules
- NEVER use a column name without verifying it exists in `DATABASE_SCHEMA.md`
- `.order()` takes ONE column per call. Chain multiple `.order()` calls for multi-column sorting.
- FK joins (`table(columns)` in `.select()`) require foreign key relationships in the database
- `form_registry` does NOT have: `sort_order`, `deadline_days`, `field_mapping`
- `generated_documents` does NOT have: `form_registry_id`, `file_name`
- `generated_documents` uses `form_library_id` (not `form_registry_id`)

## Project Structure

- **Active pages:** All in `src/pages/` directory, imported by `src/App.jsx`
- **Dead code:** Cleaned up — 64 dead files (21,394 lines) removed in commit `7e3b145`
- **Shared libs:** `src/lib/store.js` (global state), `src/lib/supabase.js` (client), `src/lib/documentService.js` (doc generation)
- **Components:** `src/components/` for shared UI components

## Tech Stack

- React 18 + Vite
- Supabase (PostgreSQL + PostgREST + Storage + Edge Functions + Auth)
- Zustand (state management via src/lib/store.js)
- react-router-dom v6
- pdf-lib (PDF generation)
- Tailwind CSS (inline styles)

## Supabase Project

- Project ref: `rlzudfinlxonpbwacxpt`
- URL: `https://rlzudfinlxonpbwacxpt.supabase.co`
- 78 tables in public schema
- 35 tables actively queried by frontend

## Known Bugs (ALL FIXED — DO NOT REINTRODUCE)

1. ~~`src/lib/documentService.js:30` - Selects non-existent columns from `form_registry`~~ **FIXED** in `ebb226a`
2. ~~`src/lib/documentService.js:33` - Orders by non-existent `sort_order` on `form_registry`~~ **FIXED** in `ebb226a`
3. ~~`src/lib/documentService.js:300-308` - Inserts wrong column names into `generated_documents`~~ **FIXED** in `ebb226a` + `7e3b145`
4. ~~`src/pages/DevConsolePage.jsx:276` - Invalid `.order('state, category')` syntax~~ **FIXED** in `ebb226a`
5. ~~`src/pages/DevConsolePage.jsx:391` - References `dealer_forms` table (doesn't exist)~~ **FIXED** in `7e3b145` (line removed)

---
> Source: [bryce-dotcom/og-dealer-app](https://github.com/bryce-dotcom/og-dealer-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
