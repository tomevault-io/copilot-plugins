## gaming-library-dashboard

> We have successfully implemented a cross-platform ETL pipeline and a unified dashboard.

# Project Context: Unified Gaming Dashboard (Updated April 10, 2026)

## Current Status: Multi-Platform Vertical Slice (Steam + Xbox)
We have successfully implemented a cross-platform ETL pipeline and a unified dashboard.

### Core Features Completed
1.  **Multi-Platform Ingestion**:
    *   `sync-steam`: Fetches 300+ raw records via official Web API.
    *   `sync-xbox`: Fetches 70+ records via OpenXBL (xbl.io) using XUID-based logic.
2.  **Advanced Normalization**:
    *   `normalize-games`: Uses fuzzy matching and title sanitization (stripping "GOTY", "Enhanced Edition", etc.) to unify raw data with IGDB metadata.
    *   Successfully handles merging duplicates across platforms (e.g., matching Elden Ring on Steam and Xbox to one canonical entry).
3.  **Frontend Dashboard**:
    *   **Responsive UI**: 4-column grid (desktop) scaling down to 1-column (mobile).
    *   **Interactive Features**: Real-time search, sorting (Alphabetical, Most Played, Most Recent).
    *   **Aggregation**: Automatically displays multiple platform badges per card and sums total playtime across all sources.
    *   **Stats Header**: Displays Total Library Count, Cumulative Playtime, and the #1 Most Played Game.

### Architectural State
*   **Database**: PostgreSQL with RLS public read policies and unique constraints on `igdb_id`.
*   **Data Pattern**: Raw JSON payloads are stored in `platform_games` before being linked to canonical `games`.
*   **Infrastructure**: Verified local execution via Deno 2.x and Supabase local stack.

## Next Steps
1.  **Xbox Playtime Stability**: Investigate why OpenXBL `/v2/titles` is missing `minutesPlayed` despite documentation, and resolve the 500 errors on specific stats endpoints.
2.  **PlayStation Integration**: Scaffold the `sync-psn` worker using reverse-engineered mobile API wrappers.
3.  **User Auth**: Transition from public read policies to authenticated user sessions for account linking.
4.  **Cloud Deployment**: Deploy Edge Functions and migrations to Supabase Production.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eag5357) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
