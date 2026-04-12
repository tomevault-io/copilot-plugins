## datadiver

> DataDiver is the Bloomberg Terminal of civic data — a real-time intelligence surface designed to show us stories before they are stories.

# DataDiver — SF Civic Data Visualization Platform

## Philosophy

DataDiver is the Bloomberg Terminal of civic data — a real-time intelligence surface designed to show us stories before they are stories.

San Francisco publishes millions of records across dozens of datasets on data.sfgov.org — emergency response times, 311 complaints, crime reports, parking citations, traffic crashes — but raw data doesn't change minds or inform decisions. A journalist shouldn't have to write SQL to notice that response times in the Tenderloin spiked 40% year-over-year. A neighborhood advocate shouldn't need a CSV to see that 311 graffiti complaints in the Mission are 2σ above the 12-month baseline. The data is public. The insight should be too.

The project's core belief is that **public data should feel alive, not archival**. Every view answers a human question: *How fast do first responders reach my neighborhood? Where do parking tickets cluster? Are 311 complaints rising or falling?* The interface should reward curiosity — click a neighborhood, zoom into a block, compare this year to last year — and surface the anomalies, trends, and patterns that become tomorrow's headlines.

### Design Principles

- **Data density over decoration.** Every pixel should convey information. Glass-card overlays on maps, inline stat cards, sidebar rankings — the UI is a lens, not a frame. No empty states, no placeholder illustrations, no loading screens that hide the structure.
- **Progressive disclosure.** Show the map and layout immediately. Skeleton shimmer where data will appear. Stat cards fade in as queries resolve. The user never stares at a blank screen wondering if something is broken.
- **Server-side truth.** Never trust a client-side sample to represent a dataset. Use Socrata's `GROUP BY` and `SUM()` to get accurate totals. A meter that earned $39.68 should show $39.68, not $4.60 from a truncated sample.
- **Temporal context by default.** A number without a trend is a number without meaning. YoY deltas on stat cards, ghost prior-year bars on charts, z-score anomaly dots on neighborhoods — every metric earns context about whether it's normal.
- **The map is the hero.** For geospatial datasets, the map occupies the full viewport. Stat cards float on glass. The sidebar is a complement, not the main event. For non-spatial datasets (911 dispatch), charts take the hero role instead.
- **Respect the data's limits.** Some datasets have geo gaps (parking citations after Oct 2025). Some have date lag. `DataFreshnessAlert` detects this and offers a one-click fix rather than showing an empty, confusing view.

### Audience

Journalists, civic researchers, neighborhood advocates, and curious residents. People who want to understand their city through its data but don't want to write SQL or download CSVs. The tool should be shareable (URL-encoded state) and exportable (PNG screenshots).

## Stack
- **Vite + React 18 + TypeScript + Tailwind v4**
- **Mapbox GL JS v3** for maps (dark-v11 basemap, `preserveDrawingBuffer: true`)
- **D3.js** for charts (histograms, heatgrids, trend charts)
- **Zustand** for global state (`src/stores/appStore.ts`)
- **React Router** with URL param sync (`useUrlSync` hook)
- **Socrata SODA API** for all SF open data (no backend)

## Project Structure
```
src/
  api/           # Socrata client + dataset registry
  components/
    charts/      # D3 chart components (ResponseHistogram, HourlyHeatgrid, PeriodBreakdownChart, etc.)
    export/      # ExportButton (html2canvas PNG)
    filters/     # DateRangePicker, CallTypeFilter, CategoryFilter, etc.
    layout/      # AppShell (sidebar nav + header)
    maps/        # MapView (Mapbox GL wrapper)
    ui/          # StatCard, Skeleton, DataFreshnessAlert, detail panels
  hooks/         # Data fetching + map interaction hooks
  stores/        # Zustand appStore
  types/         # TypeScript interfaces (datasets.ts, trends.ts)
  utils/         # Colors, time formatting, geo helpers
  views/         # 7 dataset views + Home
```

## Key Conventions

### Data Fetching
- All data comes from Socrata SODA API via `fetchDataset()` in `src/api/client.ts`
- `useDataset` hook wraps fetch with loading/error/refetch state
- **Aggregation queries**: `fetchDataset` auto-skips `defaultSort` when `$group` or aggregate functions (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`) are detected in `$select` — ordering by a non-selected field causes Socrata 400 errors
- Use **server-side aggregation** (`GROUP BY`, `SUM()`, `COUNT()`) over client-side aggregation of sampled rows — sampling produces inaccurate per-entity totals
- Dataset config in `src/api/datasets.ts` — each has `id`, `endpoint`, `dateField`, `defaultSort`

### Maps
- `MapView` calls `onMapReady` immediately (no waiting for Mapbox events)
- `useMapLayer` uses try-catch with setTimeout retry — the ONLY reliable pattern with Mapbox GL v3 + React
- Container sizing: use `w-full h-full` NOT `absolute inset-0` — Mapbox overrides position to relative
- Heatmap colors must be bright (cyan/red) on dark-v11 basemap — dark blues are invisible

### Loading States
- **Progressive skeleton loading** — no full-screen blockers
- Each component shows its own skeleton via `src/components/ui/Skeleton.tsx`
- Map area: `MapLoadingIndicator` (corner pill, not overlay)
- Stat cards: `SkeletonStatCards` in the same absolute position
- Sidebar: `SkeletonSidebarRows`, `SkeletonBreakdownList`
- Charts: `SkeletonChart`

### Trend Infrastructure
- `useTrendBaseline` hook fires 3-5 parallel Socrata queries for YoY comparison, z-score baseline, sub-period breakdown
- `PeriodBreakdownChart` renders D3 bars with ghost prior-year series
- `StatCard` accepts optional `yoyDelta` and `zScore` props
- `useDataFreshness` detects stale date ranges via `MAX(dateField)`, `DataFreshnessAlert` offers auto-adjust

### Views Pattern
Each view follows the same structure:
1. `useDataset` for raw data (map points)
2. Server-side aggregation queries for accurate stats
3. `useDataFreshness` + `DataFreshnessAlert`
4. `useTrendBaseline` for YoY/z-score
5. Map-centric (MapView + sidebar) or chart-centric (Dispatch911)
6. Skeleton loading per component zone

## Deployment
- **Vercel** auto-deploys from `main` branch
- Env vars: `VITE_MAPBOX_TOKEN`, `VITE_SOCRATA_APP_TOKEN` (set via Vercel dashboard or `vercel env add --value`)
- SPA routing: `vercel.json` has `rewrites` for `/(.*) → /index.html`
- Build: `pnpm build` → `tsc -b && vite build`
- Dev: `pnpm dev` (port 5174)

## Fonts
- Playfair Display (headlines, `.font-display`)
- Inter (body)
- JetBrains Mono (data values, `.font-mono`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sfbay)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sfbay)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
