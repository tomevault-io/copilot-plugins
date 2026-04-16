## minimal-metrics

> Minimal Metrics is a privacy-first analytics dashboard alternative to Google Analytics. Built with Node.js, SQLite, and vanilla JavaScript.

# Minimal Metrics - Cursor AI Rules

## Project Context
Minimal Metrics is a privacy-first analytics dashboard alternative to Google Analytics. Built with Node.js, SQLite, and vanilla JavaScript.

## Tech Stack
- Node.js 18+ with ES modules
- SQLite (better-sqlite3) with WAL mode
- Vanilla JavaScript (no frameworks)
- Terser for minification
- Docker/docker-compose deployment

## Architecture Components

### 1. Tracking Script (`tracker/`)
- Ultra-lightweight (<2KB minified)
- Privacy-first: no cookies, respects DNT, session-only
- Beacon API with XHR fallback
- **Always rebuild after changes**: `npm run build:tracker`

### 2. Server Backend (`server/`)
- `index.js` - HTTP server with routing
- `api/collect.js` - Batched writes (5s intervals)
- `api/stats.js` - Analytics with time periods
- `api/export.js` - JSON/CSV exports
- `db/queries.js` - Session hashing, database ops
- `db/schema.sql` - Schema with indexes
- `utils/` - Rate limiting, geo resolution

### 3. Dashboard (`dashboard/`)
- Single-page responsive design
- Dark/light theme with CSS custom properties
- Real-time updates (30s polling)
- Data export functionality

## Privacy Architecture
- IP addresses SHA-256 hashed with daily salt rotation
- sessionStorage only, no cookies/localStorage
- Raw events aggregated hourly, cleaned after 24h
- Geographic data from proxy headers only
- GDPR compliant, no personal data

## Code Style
- Use ES modules (`import`/`export`)
- Synchronous database operations
- In-memory rate limiting
- Descriptive variable names
- Comprehensive error handling
- Console logging for important events
- No external frameworks for dashboard

## Development
```bash
npm run dev          # Dev server with auto-restart
npm run build:tracker # Build tracking script
npm run init:db      # Initialize database
npm start           # Production server
```

## Environment
Copy `.env.example` to `.env`:
- `DATABASE_PATH` - SQLite file path
- `PORT` - Server port (default: 3000)
- `HOST` - Server host (default: 0.0.0.0)
- `RAW_DATA_RETENTION` - Hours to keep raw events (default: 24)
- `RATE_LIMIT_*` - API rate limits

## API Design
All stats endpoints support `?period=1h|24h|7d|30d|90d`:
- `POST /api/collect` - Tracking data collection
- `GET /api/stats/realtime` - Active visitors
- `GET /api/stats/overview` - Dashboard summary
- `GET /api/stats/pages` - Top pages (optional `limit`)
- `GET /api/stats/referrers` - Top referrers (optional `limit`)
- `GET /api/stats/countries` - Geographic stats
- `GET /api/export` - Export with `type` and `format` params

## Data Flow
1. Tracker collects minimal data (URL, referrer, session ID)
2. Beacon API sends to `/api/collect`
3. Server hashes session IDs with IP
4. Events queued in memory, flushed every 5s
5. Hourly aggregation, cleanup after 24h

## Key Constraints
- Tracking script MUST stay <2KB minified
- Database uses WAL mode for concurrency
- Rate limiting is in-memory (not persistent)
- Always rebuild tracker after modifications
- Use synchronous DB operations only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cotyledonlab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
