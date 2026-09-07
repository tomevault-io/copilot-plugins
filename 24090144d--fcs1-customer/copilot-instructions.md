## fcs1-customer

> Shared instruction set for all AI coding agents (Claude Code, Codex, Gemini CLI, etc.).

# FCS1 Customer — Agent Guide

Shared instruction set for all AI coding agents (Claude Code, Codex, Gemini CLI, etc.).
Read this before writing any code. See `CLAUDE.md` for deep technical patterns (Highcharts/i18n/config-panel details) — still agent-agnostic despite the filename. This project's primary day-to-day development has moved to Codex; this file (`AGENTS.md`) is the authoritative shared starting point for any agent.

---

## Current State

| Key | Value |
|---|---|
| Version | **v1.1.31** (released 2026-07-24) |
| Branch | `main` |
| Local dev | `npm run dev` → `http://localhost:3010` |
| Previous version | v1.1.30 |

**Local-only testing rule:** only test against localhost (`npm run dev`, port 3010). Never push, deploy, or commit unless the user explicitly asks in that turn — a past approval is not standing permission.

---

## Start Here

- Read this file (`AGENTS.md`) first, regardless of agent.
- Read `CLAUDE.md` for deep technical patterns (Highcharts drilldown gotchas, i18n conventions, config-panel wiring).
- Read `docs/co-dev-handoff.md` when resuming CO-specific work.
- Use `main` as the only release branch unless the user explicitly requests a temporary branch or worktree.
- No Claude-specific browser-preview tooling exists outside Claude Code. To verify UI changes in another agent: start the dev server (`npm run dev`), open `http://localhost:3010` in a regular browser, and manually navigate the sidebar (chain → module → hotel/corp) to the affected view. `npx tsc --noEmit` remains the authoritative compile gate on Windows regardless of agent (`next build` fails locally on Windows due to an unrelated symlink issue — see CLAUDE.md).

---

## Section Structure: KPI / Simple Charts / Long Charts (+ Scope-Specific Tables)

All four modules (CO first, then JO/MO/IM) share these three base sections, in this order:

1. **KPI** — `kpi-grid mt-0 grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-3`
2. **Simple Charts** — `chart-grid mt-5 grid grid-cols-1 md:grid-cols-2 gap-4` (2 per row) — everything lives here by default
3. **Long Charts** — `chart-grid-long mt-5 grid grid-cols-1 gap-4` (1 per row) — reserved for deep multi-level drilldowns; membership is opt-in per chart id, moved in only on explicit request

Explicit scope-specific analytical sections may follow Long Charts. Corp IM has `Hotel → Department → Category → Incident → Detail`; Corp JO has `Hotel → Department → Category → Service Item → Detail`; Corp MO has `Hotel → Department → Category → Defect → Detail`; and Corp CO has `Hotel → Cleaning Type → Stay Status → Attendant → Detail`. Hotel variants omit the Hotel level. The corresponding live routes are `/api/dashboard/im-table`, `/api/dashboard/jo-table`, `/api/dashboard/mo-table`, and `/api/dashboard/co-table`. The first level renders inline, deeper levels use a document-level modal, and every level has an icon-only CSV export. Keep each table date/hotel-filter aware and module-independent.

Each section is headed by a shared `SectionHead` component (label + horizontal rule). **`SectionHead` must include `mb-3`** on its wrapping div — this is the only thing separating the label from the cards below (grids use `mt-0`/`mt-5`, not a top margin). `CoDashboardView.tsx` and `DashboardClient.tsx` each keep their **own copy** of `SectionHead` — if you touch one, mirror the change in the other.

Long-Charts membership lives in per-module `Set<string>` constants (`MO_LONG_CHART_IDS`, `JO_LONG_CHART_IDS`, `IM_LONG_CHART_IDS` in `DashboardClient.tsx`; `LONG_CHART_IDS` in `CoDashboardView.tsx`), all currently empty. Move a chart id into the set + bump its list cap to `N = 50` only when the user names that specific chart — never batch-move charts speculatively.

---

## Module Scope

| Code | Full Name |
|---|---|
| `IM` | Incident Management |
| `JO` | Job Order |
| `MO` | Maintenance Order |
| `CO` | Cleaning Order / ACSR |

---

## Recent Version History

| Version | Date | Summary |
|---|---|---|
| **v1.1.31** | 2026-07-24 | Extended Hotel daily drilldown tables with a business grouping before each rank distribution: JO starts at Department, MO at Department, IM at Department, CO-ACSR at Cleaning Type, and CO-IR at Inspection Status. Group filters now persist through item, daily, detail, breadcrumb, and CSV export levels while Corp hierarchies remain hotel-first. |
| **v1.1.30** | 2026-07-24 | Removed the CO-IR `Duration Source` column from Corp and Hotel daily-inspector detail tables and CSV exports. Hardened the shared terminal detail renderer against PostgreSQL numeric values returned as strings, preventing client-side exceptions when formatting inspection score, credit, duration, standard, and variance. |
| **v1.1.29** | 2026-07-23 | Added live database-backed daily drilldown tables across Corp and Hotel JO, MO, IM, CO-ACSR, and CO-IR. New table routes provide rank distributions, named item/attendant/inspector drilldowns, ascending daily summaries, compact source-record details, CSV export, active dashboard filters, and literal source timestamp preservation without requiring CSV re-upload. |
| **v1.1.28** | 2026-07-23 | Standardized CO-IR, IM, and MO terminal drilldown combinations across corp and hotel renderers. CO-IR now uses Total Credit and Pass Rate columns with an Average Duration spline; IM uses Total Incident and Repeat Rate columns with Average Duration and Closing Rate splines; MO uses Total Order and Delay Rate columns with a Completed Duration spline. Shared metric-leaf helpers enforce consistent deep-teal, brick-red, burnt/amber-orange, and muted-purple series colors plus visible unit-aware data labels. |
| **v1.1.27** | 2026-07-23 | Fixed Corp IM `cim-21` no-data rendering by enriching the live database summary with the missing hotel-wide `all/ALL` incident-item aggregate. Added shared Highcharts drilldown axis-state restoration so JO/MO/CO/CO-IR/IM charts restore the correct x-axis type, labels, title, and visibility when users navigate from a combo leaf back to any earlier level or the root. |
| **v1.1.26** | 2026-07-23 | Refreshed Configuration and My Dashboard registries for JO/MO/CO-ACSR/CO-IR/IM, added independent KPI/chart/table visibility controls, improved reset-by-hotel labels, and restored Corp IM `cim-21` as the Hotel → Repeat Rate Dist → Incident Dist drilldown while assigning Hotel Performance Benchmark to `cimt-02`. |
| **v1.1.25** | 2026-07-22 | Redesigned Corp and Hotel CO-IR charts as date-first multi-level drilldowns with dynamic inspector/cleaner rank ranges and three-series performance leaves (total credit, average duration, pass rate); added Hotel COIR-11 and COIR-12 to the one-per-row Long Charts section for Room Status and Inspection Status analysis. |
| **v1.1.24** | 2026-07-22 | Added the independent CO Inspection Report (`CO-IR`) upload and dashboard pipeline, hotel/corp navigation, KPI benchmarks, multi-level charts, long charts, draggable inspector drilldown tables, reset scopes, four-language labels, and the idempotent `co_ir_records` Neon schema migration. |
| **v1.1.23** | 2026-07-22 | JO, MO, CO, and IM detail-table date/time columns now use the compact, timezone-aware `DD/MM/YY HH:mm` format. Detail-table minimum widths were reduced to match the shorter values, including separate CO widths for Stay Status and Inspector hierarchies. |
| **v1.1.22** | 2026-07-22 | Added a second, independent Inspector drilldown table to Corp and Hotel CO while preserving the existing Stay Status hierarchy. Corp drills Hotel → Cleaning Type → Inspector → Attendant → Detail; Hotel starts at Cleaning Type. Every level includes icon-only CSV export, inspector counts/details, and blank inspector values fall back to `Inspector`; labels were added across all four languages. |
| **v1.1.21** | 2026-07-22 | Added live, filter-aware analytical Table sections for Corp and Hotel IM/JO/MO/CO. Each module has its own database route and drilldown hierarchy, renders the root summary inline, opens deeper levels in a modal, exports every level to CSV, and includes four-language labels. CO adds benchmark flag icons to cleaning-record details. |
| **v1.1.20** | 2026-07-22 | Hotel IM `im-11` through `im-28` redesigned into the current multi-level drilldown pattern, ending in dual-axis combo leaves for incident count, average duration, repeat rate, and closing rate. The shared `im_dim_item_stats_map` pipeline now includes profile type, incident status, guest name, created by, and hotel-wide `all` dimensions; dashboard fetch and `im-scope` response generation must remain in lockstep. i18n was updated across all four languages and the full batch was verified through live drilldown clicks. |
| **v1.1.19** | 2026-07-17 | Primary/secondary Neon failover support added through `NEON_DB_SLOT`, `DATABASE_URL_SECONDARY`, and `DATABASE_URL_UNPOOLED_SECONDARY`. Database selection is resolved centrally by `lib/db/supabaseCompat.ts` and `lib/supabase/server.ts`, allowing a Vercel customer deployment to switch Neon projects by environment configuration and redeploy without code changes. |
| **v1.0.91** | 2026-07-08 | Default app UI theme changed from Vintage to **Chromatic Ink Wash** (`chromatic-ink`) — affects fresh sessions/browsers only (`components/layout/ThemeProvider.tsx` initial `useState`); users with a stored `fcs1-theme` localStorage preference are unaffected |
| **v1.0.90** | 2026-07-08 | IM long-drilldown follow-up release: corp IM `cim-22`..`cim-26` and hotel IM `im-41`..`im-45` now resolve drilldown axis labels from the active level, use incident item names from database-backed summary maps instead of blank fallback buckets, and keep 24-hour-distribution charts aligned with the configured organization timezone; CO/IM fetch/meta plumbing updated so dashboard views receive timezone context consistently; i18n updated across all four languages |
| **v1.0.89** | 2026-07-07 | JO/MO/IM restructured to the same KPI / Simple Charts / Long Charts section pattern CO already had (see "Section Structure" above); sub-headers ("Executive Charts", "Drilldown charts", "Chain Comparison", "Performance Gauges", "Corp Comparison Top 10", "Builder Charts", etc.) removed, charts flattened into one Simple Charts grid per scope; new `MO_LONG_CHART_IDS`/`JO_LONG_CHART_IDS`/`IM_LONG_CHART_IDS` sets (empty) + `splitLongCharts` helper in `DashboardClient.tsx`; every hardcoded chart-list cap (`topN`/`.slice(0, N)`) in JO/MO/IM builder code normalized to `N = 24`; fixed spacing bug where `DashboardClient.tsx`'s `SectionHead` was missing `mb-3` vs `CoDashboardView.tsx`'s copy; `AGENTS.md`/`CLAUDE.md` synced and repositioned for a Codex-primary workflow |
| **v1.0.88** | 2026-07-04 | JO chart redesigns (hotel + corp): cjo-02 → 2-level column drilldown (Job Status → 24-Hour Distribution); cjo-15 → 2-level drilldown (Job Status → Completed Duration Distribution, new `jo_status_dur_bkt_map`); cjo-27 ↔ cjo-03 content swap; hotel jo-01 → hour → top-10 delayed items drilldown (new `jo_hour_delayed_item_map`); hotel jo-02 category/grid chart display-code fix; fix `/api/ai/charts/list` 500 (`created_at.localeCompare` on pg `Date` → `.getTime()`); i18n all 4 langs |
| **v1.0.87** | 2026-06-27 | Theme picker rebuilt as card layout; **Color Ink Wash** replaced with **Jade & Ink** (`jade-ink`) — jade-green sidebar, rice-paper surfaces, jade/gold/deep-blue palette; `AppThemeOption` type + `getThemeSwatches()` added to `lib/theme.ts` |
| **v1.0.86** | 2026-06-27 | Two new app UI themes: **Chromatic Ink Wash** (`chromatic-ink`) and **Color Ink Wash** (`color-ink`), both light+dark variants; `lib/theme.ts` updated |
| **v1.0.85** | 2026-06-27 | Hotel IM im-04 redesign: VIP vs Non-VIP → 24-Hour Distribution column-drilldown; new `im_vip_hour_map`; `scripts/backfill_im_vip_hour_map.mjs`; i18n all 4 langs |
| **v1.0.84** | 2026-06-26 | 24-hour distribution timezone fix: backfill scripts now use `localHour(d, orgTimezone)` / `AT TIME ZONE org_timezone` instead of system-local/hardcoded UTC; local DB re-backfilled (7 hotels, Asia/Hong_Kong) |
| **v1.0.83** | 2026-06-26 | Hotel JO jo-02 redesign: Top Service Item Category → 24-Hour Job Distribution column-drilldown; new `jo_cat_hour_map`; i18n all 4 langs |
| **v1.0.82** | 2026-06-26 | Hotel JO jo-06 redesign: Job Status by 24-Hour Job Distribution bar-drilldown from `jo_status_hour_map`; i18n all 4 langs |
| **v1.0.81** | 2026-06-25 | Hotel JO jo-01/jo-03 client-side redesigns (24-Hour Delayed Distribution; Top Service Items → Completed Duration Distribution, new `jo_item_dur_bkt_map`); removed v1.0.70 jo-01↔jo-05 swap; i18n all 4 langs |
| **v1.0.80** | 2026-06-20 | JO/MO/CO chart footer notes annotated with Good/Watch/Bad or Healthy/Warning benchmark lines, all 255 notes × 4 langs, via idempotent `scripts/annotate_chart_benchmarks.mjs`; system-settings save now also routes through POST (Vercel blocked bare PUT) |
| **v1.0.79** | 2026-06-20 | Fix Configuration > System timezone save: removed `updated_at` from organizations UPDATE (column absent in Neon production) |
| **v1.0.78** | 2026-06-20 | Corp MO cmo-09/10/11 redesigned to mirror hotel mo-09/10/11 for chain data (duration distribution, 24-hour distribution, top-10 >24h defects), each with per-hotel drilldown; i18n all 4 langs |
| **v1.0.73** | 2026-06-19 | Hotel MO mo-01/mo-02 redesigned as donut drilldowns (Top 10 Category by Status; Status by Department); new `cat_status_map`; `scripts/backfill_mo_cat_status_map.mjs`; i18n all 4 langs; corp cmo-* untouched |
| **v1.0.72** | 2026-06-19 | Hotel MO charts rebuilt: `buildHotelMoCharts` emits real mo-01..mo-12 client-side (replaces legacy im-46..im-69 leak from `buildImJson`); fixes hotel + My Hotel dashboards; world map now loads for mo-06 |
| **v1.0.71** | 2026-06-19 | My Hotel MO charts fix: positional fallback in embed mode when stored MO data has legacy im-NN ids; cim-20 dual-axis column+line; gauge color/border tweaks |
| **v1.0.69** | 2026-06-13 | cjo-07 → treemap "Top Service Items (Chain)": aggregates item_map across all hotels, top 30 tiles, useHTML labels; i18n all 4 langs |
| **v1.0.68** | 2026-06-13 | cjo-07 xAxis `type:'category'`: drilldown X axis shows top 10 service item names (was inheriting hotel codes) |
| **v1.0.67** | 2026-06-13 | jo-11 primary xAxis `type:'category'` fix — drilldown dates replace item names on Y axis correctly |
| **v1.0.66** | 2026-06-13 | jo-11 drilldown changed to `bar` type — dates on Y axis, count on X axis; label updated to "Daily Job Orders" |
| **v1.0.65** | 2026-06-13 | jo-11 in-place ordering (injected charts replace stored counterpart at original slot) + date-filter support via `jo_item_date_map` (all-time fallback when map absent); FULL PERIOD badge suppressed for jo-11 when map present; `scripts/backfill_jo_item_date_map.mjs` backfills legacy rows from `jo_records` |
| **v1.0.64** | 2026-06-13 | jo-11 always injected: removed `if (idm)` guard; drilldown = daily trend when `jo_item_date_map` present, else dept breakdown from inverted `dept_item_map` — works without re-upload |
| **v1.0.63** | 2026-06-13 | jo-11 client-side injection: `jo_item_date_map` in `HotelSummary` + finalize summary; replaces stored plain-bar jo-11 with drilldown when summary data present |
| **v1.0.62** | 2026-06-13 | jo-11 redesigned: bar-drilldown "Top Service Items → Daily Trend"; new `itemDateMap` (item→date→count) accumulator in finalize route; takes effect on re-upload |
| **v1.0.61** | 2026-06-13 | cjo-07 redesigned: "Reassignment Rate by Hotel" → bar-drilldown "Top Service Items by Hotel"; primary = total jobs per hotel, drilldown = top 10 `item_map` entries |
| **v1.0.60** | 2026-06-12 | MO hotel KPI list trimmed 12 → 10 (removed `mo_unique_assets`/`mo_daily_average`); order aligned with hotel dashboard |
| **v1.0.59** | 2026-06-12 | MO corp KPI label fix: `dash-config-defs.ts` corp MO `labelPath`/`notePath` corrected from `hmo_kpi_labels` → `cmo_kpi_labels`; Corp KPI Group now shows proper names |
| **v1.0.58** | 2026-06-12 | IM corp KPI order: `corp_kpi_09` (Total Incident Volume) → position 1 (`cim_kpi_01`); `corp_kpi_01` (Corporate Risk Score) → position 9 (`cim_kpi_09`) |
| **v1.0.57** | 2026-06-12 | Config panel JO/MO/CO/IM tabs: "KPI Group" → "Hotel KPI Group" + "Corp KPI Group"; `GroupPanel.displayCodeOf` prop adds sequential codes `{mod}_kpi_01..N` / `c{mod}_kpi_01..N`; charts show hyphen-normalized IDs `jo_01..N` |
| **v1.0.56** | 2026-06-12 | My Dashboard config: corp picker shows `cjo_kpi_01..10` / `cmo_kpi_01..10` etc. (`kpiDisplayCode` scope-aware); chart codes normalised `cjo_01..28` (`chartDisplayCode`); display-only, stored keys unchanged |
| **v1.0.55** | 2026-06-12 | IM KPI alias fix: `dash-config-defs.ts` hotel IM KPI list trimmed to 10 rendered ids; aliases sequential im_kpi_01–10 (hotel) / im_kpi_11–20 (corp); `IM_HOTEL_KPI_IDS` matches |
| **v1.0.54** | 2026-06-12 | My Hotel/Corp date filter: "ALL" → "Reset"; Reset sets `applied = null` (no filter) and blank inputs |
| **v1.0.53** | 2026-06-12 | My Hotel config multi-hotel: checkbox chips replace single dropdown; `hotels: string[]` in `MyDashboardConfig`; sidebar shows one link per hotel; legacy `hotel` string auto-migrates |
| **v1.0.52** | 2026-06-12 | My Dashboard scope binding: My Hotel config requires Hotel selection; My Corp chain-only; sidebar link carries hotel; My Hotel defaults to blank date filter, CO unscoped |
| **v1.0.51** | 2026-06-12 | My Hotel CO/MO data fixes: CO works without stored dashboard JSON (null-data shell from coRows); CO sub-property hotel-code fallback (prefix match); Date-object coercion for `created_date`; legacy MO `chart_NN` → `mo-NN` id rename at render |
| **v1.0.50** | 2026-06-11 | My Hotel: hotel filter dropdown removed; `useRouter` / `router` dead code cleaned from `MyDashboardClient.tsx` |
| **v1.0.49** | 2026-06-11 | Fix My Dashboard publish: persist moved out of `setCfg` updater (impure-updater side effect could be dropped by React, losing the publish) |
| **v1.0.48** | 2026-06-11 | My Dashboard uniform KPI codes (`{mod}_kpi_NN` aliases for display + storage, auto-migration of saved configs, alias→native id resolution at render) |
| **v1.0.47** | 2026-06-11 | My Corp pooled layout: shared date-range bar + single pooled KPI/chart grids (same as My Hotel); corp chart arrays wired into embed mode; per-module corp sections removed |
| **v1.0.46** | 2026-06-11 | My Hotel pooled layout: shared date-range bar across modules, department filter removed, all KPIs/charts grouped into single grids via `MyDashEmbed` fragment mode in dashboard components |
| **v1.0.45** | 2026-06-11 | My Dashboard feature: Configuration → My Dashboard tab (compose My Hotel/My Corp from JO/MO/CO/IM items, max 10 KPI + 20 charts, drag-n-drop, chain-bound, publish to sidebar); `/my-dashboard` page renders selections through real dashboard components via `myDash` override; dashboard fetchers moved to `lib/dashboard-fetch.ts` |
| **v1.0.44** | 2026-06-11 | Dashboard Builder moved from sidebar to Configuration → Builder tab; sidebar link removed; `PlaygroundClient` rendered via `dynamic` import in `app/configuration/page.tsx` |
| **v1.0.43** | 2026-06-09 | Dashboard Builder: CSV filename parsed as `[Chain]-[Hotel]-[HotelName]-[Module]-[Country]-[DataRange]`; grouped Chain › Module › Hotel; `sourceLabel()` helper formats as readable subtitle |
| **v1.0.42** | 2026-06-09 | Dashboard Builder: Data Source uses original upload CSV filenames; new `/api/ai/charts/datasources` route; grouped Chain › Module; selecting source sets `activeOrgId` for data scoping |
| **v1.0.41** | 2026-06-09 | Dashboard Builder: Data Source selector — loads hotel+module combos from nav API on mount; selected source auto-sets module, shows as chart subtitle in sample preview and generated charts |
| **v1.0.40** | 2026-06-09 | Dashboard Builder: 3 preview themes — Vintage, Modern, Executive; theme toggle buttons in builder panel; `applyBuilderTheme` applies palette/font/axis colors; gauge/heatmap use per-theme colors |
| **v1.0.39** | 2026-06-09 | Dashboard Builder: title/subtitle; field hint modals per module (JO/MO/CO/IM) + Chart Types pop-up |
| **v1.0.38** | 2026-06-09 | Dashboard Builder: hotel/corp templates show chart type; sample preview on template select |
| **v1.0.37** | 2026-06-09 | Dashboard Builder: 3-group templates (KPI/Hotel/Corp) per module; module toggle buttons; all chart IDs aligned with configuration panel |
| **v1.0.36** | 2026-06-09 | cjo-13/cjo-14 redesign: vertical bar-drilldown Completed/Timeout Status by Hotel → 24-Hour distribution; new `jo_hour_timeout_map` accumulator in finalize route |
| v1.0.35 | 2026-06-09 | cjo-12 redesign: vertical bar-drilldown Delayed Status by Hotel → 24-Hour Delayed Job Distribution; new `jo_hour_delayed_map` accumulator in finalize route |
| v1.0.34 | 2026-06-09 | Fix config tab active indicator — overflow-x:auto clipped margin-bottom:-3px; replaced with absolute span (bottom:0, z:2) |
| v1.0.33 | 2026-06-09 | Force-redeploy all customer Vercel projects (fcs1-hk/mo/cn/my/neon) after GitHub webhook stall |
| v1.0.32 | 2026-06-09 | Sidebar auto-refreshes after any DB reset — `fcs1:nav-refresh` custom event dispatched on reset success |
| v1.0.31 | 2026-06-09 | Remove leftover password hint from ResetPanel placeholder |
| v1.0.30 | 2026-06-09 | Reset by Hotel fix: hotel list → dropdown from dashboard meta; API uses `hotel_code` not `org_id` |
| v1.0.29 | 2026-06-09 | Config tab bar height/indicator size tweaks |
| v1.0.28 | 2026-06-09 | Reset by Hotel — new panel with password gate, org+module select, upload history preview, VACUUM ANALYZE |
| v1.0.27 | 2026-06-09 | Reset Database enhanced — per-module scope, two-step preview with row-count + disk-size, TRUNCATE + VACUUM |
| v1.0.26 | 2026-06-09 | Corp JO KPIs fixed; jo-28/cjo-28 → Overdue Jobs by Category → 24-hour drilldown; duplicate toolbar buttons removed |
| v1.0.25 | 2026-06-08 | BV config panel for all modules; `lib/dash-config-defs.ts` committed; cco_chart_14 → treemap; cco_chart_21 → manual click-to-treemap |

---

## Key Patterns Established (v1.0.26+)

### Bar-drilldown charts (JO corp: cjo-12, cjo-13, cjo-14)
- Always use **vertical `column`** for both primary and drilldown series — never `bar` (horizontal)
- Primary colour: `#0F766E` (green); drilldown colour: `#C2410C` (orange)
- Highcharts `drilldown:` key on data points, matching `id` in `drilldown.series`
- 24-hour x-axis on drilldown: `Array.from({ length: 24 }, (_, i) => i)` → `"HH:00"` labels

### Stored-summary data fallback
New accumulator fields added to `finalize/route.ts` won't appear in stored DB summaries until data is re-uploaded.
Always identify an equivalent older field to derive the data from:
- `jo_hour_delayed_map` (v1.0.35) → derive from `jo_overdue_cat_hour_map` by summing across categories
- `jo_hour_timeout_map` (v1.0.36) → derive from `jo_status_hour_map` filtered by `status.includes('timeout')`
- `jo_hour_comp_map` (existing) → used directly for cjo-13

---

## Core Rules

- Keep each module independent. Do not let corp-level changes overwrite hotel-level charts, labels, routes, or formulas.
- Preserve the existing architecture and UI patterns. Reuse shared layout pieces, but keep module-specific chart registries, KPI formulas, and notes separate.
- When changing dashboard logic, update both corp and hotel variants only if the module already supports both.
- Use local time for dashboard calculations and labels unless a file explicitly requires a different timezone.
- Keep parsing null-safe. Missing dates, status, duration, room number, floor, or attendant values must not crash the dashboard.
- Prefer fallback calculations and empty states over hard failures.

---

## Module Structure

- `IM`, `JO`, and `MO` remain the legacy dashboard modules.
- `CO` uses the cleaning-order ACSR model and must keep its own schema, upload parsing, KPI formulas, chart ids, and benchmark table logic.
- Corp charts must use corp-specific ids and corp data pipelines. Hotel charts must use hotel-specific ids and hotel data pipelines.
- Do not reuse hotel chart ids inside corp views, or vice versa.

---

## UI / UX Expectations

- Keep sidebar routes, breadcrumb titles, toolbar controls, filters, KPI cards, charts, notes, and export actions aligned with the active module.
- Maintain the existing 4-language i18n structure. If a visible label changes, update all four language files together.
- Dark/light mode must stay synchronized with the document class and the dashboard state.
- Keep Highcharts as the charting system for dashboard analytics.
- Treemap labels always require `useHTML: true` when the format string contains HTML tags.

---

## Highcharts Patterns

### Do NOT use drilldown module for column → treemap
The Highcharts drilldown module crashes (`TypeError: Cannot read properties of undefined (reading 'x')` in `getTitlePosition`) when the drilldown series type is `treemap` and the parent chart is cartesian. Use `plotOptions.column.point.events.click` + `chart.addSeries()` instead (see `CLAUDE.md` for the full pattern).

### Treemap data uses `value`, not `y`
```js
{ name: 'Alice', value: 14 }  // ✅
{ name: 'Alice', y: 14 }      // ❌ (y is for cartesian series)
```

---

## Data / Schema Expectations

- Treat SQL migrations in `sql/migrations/` as the source of truth for schema changes.
- Make migrations idempotent when possible.
- For CO uploads, preserve cleaning-order semantics and business-value fields such as completion rate, duration, on-time rate, re-clean rate, inspection pass rate, and productivity metrics.

---

## Validation

- After code changes, run `npx tsc --noEmit` — this is the authoritative compile check on Windows (local `next build` fails due to a Windows webpack symlink issue unrelated to code correctness).
- Validate all 4 i18n JSON files: `node -e "['en','ja','zh-TW','zh-CN'].forEach(l => { try { JSON.parse(require('fs').readFileSync('i18n/'+l+'_lang.json','utf8')); console.log('OK',l); } catch(e) { console.log('FAIL',l,e.message); } })"`
- For dashboard changes, verify module routing, empty states, KPI formulas, chart rendering, and language switching.
- For schema changes, confirm the migration applies cleanly to the target Postgres (Neon) database.
- Before claiming a release is complete, verify the intended Vercel project is linked to `main` and the target customer database matches the requested customer code.
- Before pushing, ensure `lib/dash-config-defs.ts` is tracked (`git add lib/dash-config-defs.ts` if untracked).
- Every time you push to GitHub/Vercel, identify and report the rollback point: the last known-good commit hash/version before the push. State both after pushing, e.g. "Pushed v1.1.9 (`1810141`); rollback point is v1.1.8 (`3c30c9d`)." Prefer `git revert <new-commit>` (safe, non-destructive, auto-redeploys via Vercel) over `git reset --hard` + force-push, which needs explicit user confirmation every time.

---
> Source: [24090144d/fcs1-customer](https://github.com/24090144d/fcs1-customer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
