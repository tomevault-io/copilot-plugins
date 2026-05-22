## ierne-snooker

> - We use a component-based approach for HTML.

# Project: Snooker League Website

## Component Architecture
- We use a component-based approach for HTML.
- **Source Components**: Located in `/components/` (e.g., `header.html`, `footer.html`).
- **Implementation**: These components are pasted directly into final `.html` files.

## JavaScript Module Architecture
- We use a modular, component-based approach for JavaScript.
- **Configuration**: Located in `/assets/js/config/` (e.g., `app-config.js` - Edge Function URL).
- **Utilities**: Located in `/assets/js/utils/` (e.g., `api-client.js`, `formatters.js`).
- **Components**: Reusable JavaScript components in `/assets/js/components/` (e.g., `knockout-renderer.js`, `league-standings.js`).
- **Pages**: Page-specific modules in `/assets/js/pages/` (e.g., `index.js`, `fixtures.js`).
- **Entry Point**: Main router in `/assets/js/main.js`.
- **No Duplication**: Common code must be extracted to utilities or components, never duplicated.

## Backend
- **Database**: Supabase Postgres, schema `ierne_snooker` on project `Apps` (`yzyipxvlsoxfphwobfkb`).
- **Schema**: `leagues`, `seasons`, `season_players`, `players`, `fixtures`, `handicaps`, `breaks`. Standings are computed live via the `league_standings_v` view from `fixtures`; head-to-head tiebreaker via `head_to_head_v`. There is no stored `league_standings` table.
- **API**: Single Edge Function `ierne-api` exposing JSON read endpoints (`getFixtures`, `getStandings`, `getHandicaps`, `getPlayers`, `getTopBreaks`). All read endpoints accept `?season=<id>` (default: row in `seasons` with `is_current = true`). Source: `supabase/functions/ierne-api/index.ts`.
- **Migrations**: SQL files in `supabase/migrations/`.
- **Data sync**: `scripts/migrate-sheets-to-supabase.mjs` reads the legacy Google Sheets CSVs and upserts into Supabase. Idempotent, safe to re-run.
- **Plans**: Architectural plans live in `docs/plans/` and stay in sync with the code.

## Rules for AI (Cursor)
1. **HTML Syncing**: Whenever I ask to update the "Header" or "Footer," always update the code in the `/components/` folder first, then propagate to all `.html` files.
2. **JavaScript Organization**: When adding or modifying JavaScript functionality:
   - Update shared utilities in `/assets/js/utils/` first
   - Update reusable components in `/assets/js/components/` for shared UI logic
   - Update page-specific modules in `/assets/js/pages/` for page logic
   - Never duplicate code - extract common patterns to utilities/components
3. **Backend Configuration**: To change the API endpoint, only update `/assets/js/config/app-config.js` - this is the single source of truth for the Edge Function URL.
4. **API access pattern**: Pages call `ApiClient.get({ action: '...' })` (defined in `assets/js/utils/api-client.js`). Never call PostgREST or supabase-js directly from the browser. New read endpoints go in `supabase/functions/ierne-api/index.ts`.
5. **CORS-safe fetches**: Use GET with no custom headers, or POST with `application/x-www-form-urlencoded`, to avoid triggering a preflight that the function doesn't handle.
6. **Schema changes**: Add a new file in `supabase/migrations/` (timestamp-prefixed) and apply with `supabase db push` or via the apply_migration MCP tool. All objects live in the `ierne_snooker` schema.
7. **Script Loading Order**: Scripts must be loaded in this order: config → utils → components → pages → main.js
8. **Relative Paths**: Always use relative paths (e.g., `assets/css/style.css`, not `/assets/css/style.css`).
9. **No absolute URLs**: Never include `http://localhost` or drive letters like `C:/Users/`.

---
> Source: [ZarleyLtd/ierne-snooker](https://github.com/ZarleyLtd/ierne-snooker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
