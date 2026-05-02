## tarmeez-app

> ╔══════════════════════════════════════════════════════════════╗

╔══════════════════════════════════════════════════════════════╗
║         ANALYTICS SYSTEM — ENGINEERING RULES (TARMEEZ)      ║
╚══════════════════════════════════════════════════════════════╝

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SYSTEM CONTEXT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Platform:   Tarmeez — multi-tenant SaaS store builder
Backend:    NestJS + Prisma + PostgreSQL + TimescaleDB
Frontend:   Next.js 16+ | React 19+ | Tailwind CSS v4
Charts:     shadcn/ui Charts (built on Recharts)
State:      RTK Query (polling every 60 seconds)
Styling:    STYLE-RULES 1-10 (semantic tokens only)
Direction:  RTL Arabic (dir="rtl")

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 1] DATA ISOLATION — NEVER MIX WITH STORE DATA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Analytics data is completely separate from store data.

NEVER:
✗ Add analytics columns to existing models (Order, Product)
✗ Query analytics from store-related services
✗ Mix analytics logic inside MerchantModule or StoresModule

ALWAYS:
✅ All analytics in dedicated AnalyticsModule
✅ Separate Prisma models for analytics tables
✅ Separate NestJS service: AnalyticsService
✅ Separate aggregation service: AggregationService
✅ Separate RTK Query API: analyticsApi.ts

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 2] PRIVACY — ANONYMOUS DATA ONLY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
No personally identifiable information (PII) is stored.

NEVER store:
✗ IP address (full or partial)
✗ User name, email, or any identifier
✗ Browser fingerprint
✗ Any data that links a visit to a specific person

ALWAYS store:
✅ sessionId — anonymous UUID generated client-side
   stored in sessionStorage only (cleared on tab close)
✅ Country and city derived from IP at request time
   then IP is discarded immediately — never persisted
✅ Device type, browser name (no version details)
✅ Timestamps, page paths, referrer domain only
   (not full referrer URL with query params)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 3] COLLECTION ENDPOINT — PUBLIC + FAST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST /api/analytics/collect is the single ingestion point.

Rules:
✅ Public endpoint — no authentication required
✅ Must return 200 in < 50ms (fire and forget)
✅ Use queue/buffer — never write to DB synchronously
   on every request (would kill DB under load)
✅ Validate storeId exists before accepting data
✅ Rate limit: max 100 requests/minute per sessionId
✅ Payload max size: 2KB
✅ Use navigator.sendBeacon on client — never fetch()
   (sendBeacon doesn't block page unload)

Queue pattern:
  Request → validate → push to in-memory buffer
  → flush to DB every 10 seconds in batch

NEVER:
✗ await prisma.create() inside collect endpoint
✗ Any heavy computation in collect endpoint
✗ Logging full request body (contains user behavior)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 4] TRACKING SCRIPT — LIGHTWEIGHT + ISOLATED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
The tracking script runs in every storefront.

MANDATORY constraints:
✅ Max size: 5KB minified and gzipped
✅ No external dependencies — vanilla JS only
✅ Must not block page rendering (async/defer)
✅ Must not use cookies
✅ Must not use localStorage (use sessionStorage only)
✅ Must throttle mousemove: max 1 event per 100ms
✅ Must throttle scroll: max 1 event per 500ms
✅ Must use sendBeacon — never fetch() or XMLHttpRequest
✅ Must handle errors silently — never throw to console
✅ Injected via Next.js Script component with
   strategy="afterInteractive"

Script receives via data attributes:
  data-store-id="uuid"
  data-endpoint="/api/analytics/collect"

NEVER:
✗ Import React or any npm package
✗ Modify DOM in any way
✗ Block main thread
✗ Send on every keystroke or mouse pixel

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 5] DATABASE — TIMESCALEDB PATTERNS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Raw event tables are TimescaleDB hypertables.
Aggregated tables are regular PostgreSQL tables.

Hypertables (raw data — high write volume):
  page_views    — partition by: time (1 day chunks)
  events        — partition by: time (1 day chunks)
  heatmap_data  — partition by: time (1 day chunks)

Regular tables (pre-computed aggregates):
  analytics_hourly  — computed every hour by cron
  analytics_daily   — computed every midnight by cron

Rules:
✅ NEVER query raw hypertables for dashboard display
   Always query aggregated tables for performance
✅ Raw tables are write-only from the API perspective
✅ Aggregation runs as NestJS @Cron job every hour
✅ Add storeId index on ALL analytics tables
✅ Add retention policy: raw data kept 90 days only
   aggregated data kept indefinitely
✅ All time columns use UTC — convert in frontend

NEVER:
✗ SELECT * FROM page_views (full table scan)
✗ Complex JOINs on hypertables in real-time
✗ Store analytics in same Prisma schema file
   as store models — use separate schema or
   clearly marked section with comment separator

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 6] MULTI-TENANCY — STORE ISOLATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Every analytics record MUST have storeId.
A merchant can ONLY see their own store's analytics.

✅ Every query: WHERE storeId = merchant.storeId
✅ MerchantGuard on all GET /merchant/analytics/* routes
✅ Verify storeId in collect endpoint matches
   a real active store before accepting data
✅ SessionId is scoped per store — same visitor
   in two stores = two different sessionIds

NEVER:
✗ Allow merchant to query another store's analytics
✗ Accept collect events for unknown storeId
✗ Expose raw sessionIds in API responses

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 7] API DESIGN — AGGREGATED RESPONSES ONLY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Analytics API never returns raw events to frontend.
Always return pre-aggregated, display-ready data.

Endpoints:
GET /merchant/analytics/overview
  → { visitors, pageViews, revenue, conversionRate,
      trend: { today, yesterday, thisWeek, thisMonth } }

GET /merchant/analytics/traffic?period=7d
  → { sources: [], devices: [], countries: [],
      hourly: [], daily: [] }

GET /merchant/analytics/pages?period=7d
  → { pages: [{ path, views, avgDuration, bounceRate }] }

GET /merchant/analytics/funnel?period=7d
  → { steps: [{ name, count, dropoffRate }] }

GET /merchant/analytics/sales?period=30d
  → { daily: [], weekly: [], monthly: [],
      topProducts: [], avgOrderValue: number }

GET /merchant/analytics/heatmap?page=/&type=click
  → { points: [{ x, y, weight }], viewport: string }

Rules:
✅ period param: 1d | 7d | 30d | 90d | 1y | all
✅ All monetary values in SAR
✅ All percentages as 0-100 numbers (not 0-1)
✅ Response cached for 60 seconds (Redis or in-memory)
✅ Max response size: 100KB

NEVER:
✗ Return individual session data
✗ Return timestamps with millisecond precision
  (use date strings: "2026-03-15")
✗ Return more than 365 data points in one response

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 8] FRONTEND — CHARTS AND DISPLAY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
All charts use shadcn/ui Charts (Recharts-based).
All colors use CSS chart variables from globals.css.

✅ Use: var(--color-chart-1) through var(--color-chart-5)
✅ All charts inside shadcn <Card> component
✅ All charts have loading skeleton state
✅ All charts have empty state (no data message)
✅ All charts have error state (retry button)
✅ Date range selector on every analytics page
✅ RTK Query polling every 60 seconds
✅ Use shadcn ChartTooltip for all tooltips

Chart types per metric:
  Sales over time     → AreaChart
  Traffic sources     → PieChart or DonutChart
  Devices             → PieChart
  Page views daily    → BarChart
  Conversion funnel   → custom BarChart (horizontal)
  Top products        → HorizontalBarChart
  Heatmap             → custom canvas overlay

NEVER:
✗ Import recharts directly — use shadcn Charts
✗ Hardcode colors (use chart CSS variables)
✗ Show raw numbers without formatting
  (1000 → "1,000" | 1500000 → "1.5M")
✗ Show charts without loading state
✗ Fetch analytics in Server Components
  (use Client Components + RTK Query only)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 9] HEATMAP — CANVAS RENDERING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Heatmap is rendered as canvas overlay on page screenshot.

Architecture:
  1. Backend returns: [{ x: float%, y: float%, weight: int }]
  2. Frontend renders canvas overlay using heatmap.js
     OR custom canvas with radial gradients
  3. Overlay is shown on top of page iframe or screenshot

Rules:
✅ Use dynamic import for heatmap rendering library
   (heavy — only load when heatmap tab is active)
✅ X and Y stored as percentages (0-100)
   not absolute pixels (responsive to viewport)
✅ Separate heatmap data per viewport:
   mobile and desktop stored and shown separately
✅ Minimum 100 data points before showing heatmap
   (show "insufficient data" message below threshold)
✅ Max data points returned per request: 10,000
   (aggregate nearby points on backend)

NEVER:
✗ Store absolute pixel coordinates
✗ Show heatmap with less than 100 data points
✗ Load heatmap library on initial page load

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 10] SALES ANALYTICS — FROM ORDERS NOT EVENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Sales data comes from the Orders table — NOT from
tracking events. Never duplicate financial data.

✅ Revenue, orders count, avg order value:
   → Query from Order model (already exists)
   → Filter by storeId and createdAt date range
✅ Top products:
   → Query from OrderItem model (already exists)
✅ Conversion rate:
   → visitors from analytics_daily
   → purchasers from Order count
   → conversionRate = (orders / visitors) * 100

NEVER:
✗ Track purchase events in the analytics pipeline
  for financial reporting (use Orders model)
✗ Show revenue numbers that differ from Orders
✗ Duplicate order data in analytics tables

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 11] AGGREGATION — CRON JOB PATTERN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pre-computed aggregates are the backbone of performance.

✅ AggregationService runs via NestJS @Cron:
   Hourly:  '0 * * * *'   — compute last hour stats
   Daily:   '0 0 * * *'   — compute yesterday stats
✅ Aggregation is idempotent:
   Running twice produces same result (upsert pattern)
✅ Aggregation processes one store at a time
   with 100ms delay between stores (avoid DB spike)
✅ Log aggregation duration and row count
✅ If aggregation fails: log error, continue next store
   Never crash the entire cron job

NEVER:
✗ Aggregate in real-time per request
✗ Run aggregation more than once per hour
✗ Block other operations during aggregation

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 12] CHECKLIST BEFORE MARKING DONE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Before marking any analytics feature complete:

BACKEND:
□ No PII stored (no IP, no name, no email)
□ collect endpoint returns 200 in < 50ms
□ storeId validated on every collect request
□ All queries use WHERE storeId = ...
□ MerchantGuard on all GET analytics routes
□ Raw tables never queried for dashboard
□ Aggregation is idempotent (upsert pattern)
□ TimescaleDB hypertable created correctly
□ Retention policy set (90 days for raw data)

TRACKING SCRIPT:
□ Size < 5KB minified
□ No cookies used
□ sendBeacon used (not fetch)
□ mousemove throttled (100ms)
□ scroll throttled (500ms)
□ Errors caught silently
□ Loaded with strategy="afterInteractive"

FRONTEND:
□ No raw event data displayed
□ All charts use shadcn Charts
□ All colors use var(--color-chart-*)
□ Loading skeleton on every chart
□ Empty state on every chart
□ Error state with retry on every chart
□ RTK Query polling every 60 seconds
□ Date range selector works correctly
□ Numbers formatted (1000 → "1,000")
□ Works in both dark and light mode
□ RTL layout correct

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STRICT WARNINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✗ Never store IP addresses or any PII
✗ Never query raw hypertables for dashboard display
✗ Never write to DB synchronously in collect endpoint
✗ Never load tracking script synchronously
✗ Never use fetch() in tracking script (use sendBeacon)
✗ Never mix analytics models with store models
✗ Never allow cross-store analytics access
✗ Never hardcode chart colors (use CSS variables)
✗ Never show charts without loading/empty/error states
✗ Never use financial data from tracking events
   (use Orders model for revenue and sales)
✗ Never return raw session data to frontend

---
> Source: [mohamed-elbarrah/Tarmeez-app](https://github.com/mohamed-elbarrah/Tarmeez-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
