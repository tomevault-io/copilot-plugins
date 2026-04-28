## leszy-run

> Race timing system. RFID readers detect participants crossing start/finish gates.

# LeszyRun — Development Guide for Claude

## Project summary

Race timing system. RFID readers detect participants crossing start/finish gates.
Events are published via MQTT. Backend processes them and stores results in local
PostgreSQL. Syncs to Supabase when online. See ARCHITECTURE.md for full design.

## Hardware documentation

- `docs/impinj-r700-api/endpoints.md` — known R700 REST API endpoints (system status, antennas, MQTT, inventory presets)
- `docs/impinj-r700-api/mqtt.md` — full `MqttConfiguration` schema with all field names, types, constraints, and common wrong-name mistakes
- `docs/impinj-r700-api/inventory-preset-fields.md` — inventory preset field reference: `transmitPowerCdbm` range, `inventorySession` values, `inventorySearchMode` options, `estimatedTagPopulation` sizing, `rfMode` IDs (note: R700 does NOT support mode 1000)

## Language and runtime

- **JavaScript only** — no TypeScript. Plain JS with JSDoc where helpful.
- Node.js backend (Fastify)
- React frontend (Vite)

## Stack decisions (do not change without discussion)

- ORM: **Drizzle** (not Prisma, not raw SQL)
- Backend framework: **Fastify** (not Express)
- Frontend build: **Vite** (not CRA, not Next.js)
- UI: **shadcn/ui** + **Tailwind v4**
- Tables: **TanStack Table v8** (inline editing)
- Server-side real-time: **WebSockets** via `ws` npm package (not Socket.io)
- Database: **PostgreSQL 16** in Docker with named volume `pgdata`
- MQTT client: `mqtt` npm package
- Supabase sync: `@supabase/supabase-js`
- PDF export: `pdf-lib`

## Monorepo structure

```
LeszyRun/
  backend/      Node.js + Fastify
  frontend/     React + Vite (admin UI)
  public/       React + Vite (public-facing: live results, volunteer bib entry, self-service check-in)
  packages/ui/  Shared UI components (@leszyrun/ui)
  mosquitto/    native macOS, NOT dockerized (hardware constraint)
```

**`packages/ui/` rule:** All race result rendering (status badges, position estimation, podium, results tables) MUST use shared components from `@leszyrun/ui`. Never duplicate result display logic in `frontend/` or `public/` — if a component is missing, add it to `packages/ui/` first, then import in both apps.

## Running locally

```bash
# Start Mosquitto (native, from project root)
/opt/homebrew/sbin/mosquitto -c mosquitto/config/mosquitto.conf

# Start everything else
docker compose up
```

Admin frontend: http://localhost:3000
Backend API: http://localhost:3001
PostgreSQL: localhost:5432

Public app (landing page + kalendarz) — run separately:
```bash
cd public && npx vite --port 3002
```
Public app: http://localhost:3002

## Environment variables

Backend (set in docker-compose.yml or .env):
- `DATABASE_URL` — postgres connection string
- `MQTT_URL` — mqtt://host.docker.internal:1883
- `PORT` — default 3001
- `SUPABASE_URL` — optional, sync disabled if missing
- `SUPABASE_SERVICE_ROLE_KEY` — service_role key (NOT anon/publishable), bypasses RLS. Backend only. Sync disabled if missing.

Frontend:
- `VITE_API_URL` — http://localhost:3001
- `VITE_WS_URL` — ws://localhost:3001

SMS (backend, optional — SMS disabled if missing):
- `SMSAPI_TOKEN` — API token for SMSAPI.pl
- `SMSAPI_SENDER` — sender name registered with SMSAPI

Scraper (backend, optional — URL resolver disabled if missing):
- `BRAVE_SEARCH_API_KEY` — API key from https://brave.com/search/api/ (1000 queries/month free tier)

## Backend conventions

- All routes in `src/routes/`, one file per resource
- Register routes in `src/server.js` with prefix (`/api/events`, `/api/participants`, etc.)
- Drizzle schema in `src/db/schema.js`, client in `src/db/index.js`
- Use Drizzle migrations (`drizzle-kit generate` + `drizzle-kit migrate`)
- **Migrations MUST be registered in `src/db/migrations/meta/_journal.json`** — Drizzle ignores SQL files not listed there. When writing a migration manually: create the `.sql` file AND add an entry to the journal (`idx`, `version: "7"`, `when`, `tag` matching filename without `.sql`, `breakpoints: true`). If a migration already ran (backend started and logged "Migrations complete"), do NOT modify the SQL file — create a new numbered file + new journal entry instead.
- **DDL changes MUST be applied to both local DB and Supabase.** Local DB uses Drizzle migrations (auto-run on backend start). Supabase must be updated separately via `mcp__supabase__apply_migration`. Every schema change (new table, alter column, add index, etc.) requires both — never apply to one without the other.
- WebSocket broadcaster in `src/ws/broadcaster.js` — export a `broadcast(event, data)` function
- MQTT client starts on server boot, crossing detector subscribes to it
- Supabase sync runs as a `setInterval` in background, does not block requests
- All IDs are UUIDs (`gen_random_uuid()` in Postgres, `crypto.randomUUID()` in JS)
- Timestamps as `timestamptz` in DB, ISO strings in API responses

## API conventions

- REST: `GET /api/:resource`, `POST /api/:resource`, `PATCH /api/:resource/:id`, `DELETE /api/:resource/:id`
- Always return `{ data }` wrapper for single items, `{ data: [] }` for lists
- Errors: `{ error: 'message' }` with appropriate HTTP status
- PATCH uses partial updates — only send changed fields
- Import endpoints: `POST /api/events/:id/import/categories`, `POST /api/events/:id/import/participants`
- Import returns: `{ imported: N, updated: N, skipped: N, errors: [{ row, message }] }`

## WebSocket events (backend → frontend)

All messages are JSON: `{ type, payload }`

| type | payload | when |
|---|---|---|
| `rfid:raw` | `{ epc, rssi, antennaPort, topic }` | every MQTT event (during scan mode or active race) |
| `rfid:crossing` | `{ participantId, gate, confirmedAt, raceRunId }` | confirmed gate crossing |
| `race:update` | `{ raceRunId, status }` | race started, stopped |
| `result:update` | `{ raceRunId, participantId, position, durationMs, status }` | new finish or position change |
| `sync:status` | `{ status, pendingCount, lastSyncedAt }` | sync worker status changes |

## Frontend conventions

- Use TanStack Query for all API calls (not raw fetch in components)
- Inline table editing: cell blur or Tab/Enter triggers `PATCH`, optimistic update, revert on error
- All forms use controlled inputs
- No page-level loading spinners — use skeleton placeholders
- shadcn/ui components only (no mixing other component libraries)
- File naming: PascalCase for components, camelCase for lib/utils

## RFID Crossing Detector

Lives in `src/mqtt/crossingDetector.js`. In-memory state only.

**Exit-triggered algorithm** — a crossing is confirmed when a tag's signal disappears for
`gone_window_seconds` (default 3 s). The confirmed timestamp is always the **peak** reading —
when the runner was physically closest to the antenna.

Works for mass starts: runners standing in the corral generate continuous readings, goneTimer
keeps resetting → no confirmation until they actually run through.

State maps:
- `inRange`: `Map<"${epc}:${raceRunId}", { peakRssi, peakTime, antennaPort, topic, goneTimer, maxTimer }>`
- `recentWindow`: dedup within 200 ms per EPC
- `startedParticipants`: `Set<participantId>` per race (in-memory, avoids DB lookup per crossing)

Flow per reading:
1. Tag not in `inRange` → create entry, arm `goneTimer` (goneWindowMs); arm `maxTimer` (fallbackMs) **only if already started** (finish crossings only)
2. Tag in `inRange`, RSSI improved → update peak; reset `goneTimer`; `maxTimer` keeps running
3. Tag in `inRange`, RSSI not improved → reset `goneTimer` only
4. `goneTimer` fires (silence for goneWindowSeconds) → `confirmCrossing(peakTime)`
5. `maxTimer` fires (fallbackSeconds elapsed, finish only) → `confirmCrossing(peakTime)`
6. 1st confirmed crossing → `gate = start`, add to `startedParticipants`
7. 2nd confirmed crossing → `gate = finish`, add to `finishedParticipants`, calc `durationMs`

See the mermaid flowchart at the top of `crossingDetector.js` for a full diagram.

Configurable per event (stored in `events` table):
- `gone_window_seconds`: default `3` s — silence window to confirm a crossing
- `fallback_seconds`: default `10` s — force-confirm timeout for **finish crossings only**
- `rfid_mode`: `'single'` (default) or `'separate'`
- `rfid_topic_main`: default `'leszyrun'`
- `rfid_topic_finish`: default `'leszyrun/finish'` (only used when `rfid_mode = 'separate'`)

## Calendar event status values (Supabase `calendar_events` table)

`pending` → `active` (admin approves via "Do przeglądu" tab)
`pending` → `rejected` (admin rejects — hidden forever, prevents scraper re-adding)

- Scraped events default to `pending` (Supabase column default)
- Manual events created via admin UI are set to `active` immediately
- Public kalendarz only shows `active` events
- Admin "Do przeglądu" tab shows `pending` events with OK/NIE/X actions
- Dedup finds rejected events by `source + source_id` but skips updating them
- URL resolver and LLM enricher process both `active` and `pending` events

## Participant status values

`registered` → `checked_in` → `started` → `finished`
                                         → `dnf`    (manual)
`registered` / `checked_in`             → `dns`    (manual)
any                                      → `dsq`   (manual, requires `status_note`)

## Race run lifecycle

- `pending` → `active` (on start) → `finished` or `cancelled`
- Multiple race_runs can exist per category (restarts create a new one)
- Previous runs are not deleted, just have status `cancelled`

## Supabase project

Project ID: `<your-supabase-project-id>`
Sync is primarily one-way: local PostgreSQL → Supabase.
Sync is disabled when `SUPABASE_URL` env var is missing.

**Reverse sync exception:** `checkins` and `checkin_documents` tables flow Supabase → local.
The public self-service check-in page and volunteer app write directly to Supabase.
A reverse sync worker (`src/sync/checkinSync.js`) polls Supabase every 30s and pulls
new/updated checkin rows into local PostgreSQL. Admin check-in from the backend also
writes to Supabase first (not local), so all check-in data has a single source of truth in Supabase.

**Supabase-only tables** (no Drizzle schema, no local migration — apply via `mcp__supabase__apply_migration` only):
- `event_secrets` — per-event check-in PINs
- `calendar_events` — aggregated race calendar from scrapers + manual entry
- `geocode_cache` — Nominatim geocoding results cache
- `url_suggestions` — Brave Search URL candidates pending admin review

## Supabase sync — how it works

The sync worker (`src/sync/supabase.js`) runs every 30s and pushes all local rows
where `synced_at IS NULL`. After a successful upsert it sets `synced_at = now()` locally.

**A Postgres trigger (`0010_sync_trigger` migration) automatically resets `synced_at = NULL`
whenever a row is updated with real data changes.** This means any UPDATE to an already-synced
row (crossing detector setting `start_time`, API updating `category_id`, etc.) will be
re-synced automatically on the next cycle.

How the trigger decides what counts as a "real" change vs. the sync worker marking a row:
- Sync worker does `SET synced_at = now()` → `OLD.synced_at ≠ NEW.synced_at` → trigger passes through
- Any other UPDATE → `synced_at` unchanged → trigger sets it to `NULL` → sync picks it up

**Rules when adding new tables or mutation paths:**
- Every new table that syncs to Supabase needs a `trg_reset_synced_at_<table>` trigger added to the migration
- You do NOT need to manually reset `syncedAt` in route handlers — the trigger handles it
- If you bypass Drizzle and write raw SQL updates, the trigger still fires automatically
- `gate_crossings` and `checkpoint_observations` are insert-only — no trigger needed there

## SMS check-in API endpoints

- `POST /api/events/:eventId/sms/checkin` — send check-in SMS to specific participants (`{ participantIds }`)
- `POST /api/events/:eventId/sms/checkin-all` — send check-in SMS to all participants with phone numbers who haven't been sent one yet
- `POST /api/participants/:id/checkin` — admin check-in (writes to Supabase, reverse sync pulls to local)
- `GET /api/events/:eventId/documents` — list event documents (acknowledgements, required docs)
- `POST /api/events/:eventId/documents` — create event document
- `PATCH /api/documents/:id` — update event document
- `DELETE /api/documents/:id` — delete event document
- `GET /api/events/:eventId/secrets/checkin-pin` — get check-in PIN from Supabase
- `POST /api/events/:eventId/secrets/checkin-pin` — generate new check-in PIN
- `POST /api/events/:eventId/sync/checkins` — trigger immediate checkin reverse sync

## Public app — Landing page & Kalendarz

The `public/` app serves two purposes:
1. **Landing page** (`/`) — leszy.run marketing site for organizers and runners
2. **Kalendarz** (`/kalendarz`) — aggregated calendar of all running/NW events in Poland
3. **Event pages** (`/events/:slug/*`) — live results, check-in, volunteer views

The landing page and kalendarz read directly from Supabase (`calendar_events` table for kalendarz, `events` table for upcoming leszy.run events). No backend API needed for these pages.

### Logo
- `public/public/logo-bez-napisu.svg` — Leszy character without text. Two green leaves (top-left, top-right), black body/roots.
- `public/public/logo.svg` — full logo with text (used as watermark in `app.css`)

## Event scraper & enrichment pipeline

Standalone scripts (not API endpoints) that scrape Polish running event websites, enrich with AI, and publish to the `calendar_events` Supabase table. See [docs/scrapers.md](docs/scrapers.md) for full pipeline documentation.

### Data sources (7 scrapers)

| Source | Events/year | Method | Data quality |
|--------|-------------|--------|--------------|
| maratonypolskie.pl | 500+ | Playwright (HTML tables) | Low (listing only, no reg URLs) |
| b4sportonline.pl | 100+ | fetch+Cheerio (AJAX pagination) | Medium (city, name, date, reg URL, some distances) |
| datasport.pl | 200+ | Cheerio (detail pages) | High (distances from h4 headings) |
| elektronicznezapisy.pl | 300-500 | Cheerio + dostartu API enrichment | Medium-High |
| biegiwpolsce.pl | 1000+ | Cheerio (paginated) | Medium (tagged distances) |
| dostartu.pl | 250+ | REST API (JSON) | **Highest** (structured classifications) |
| timekeeper.pl | 50-150 | Cheerio (internal events only) | Good (regulamin PDFs, organizer sites) |

### Pipeline architecture

```
scraper_* tables → scraper_all → calendar_events
        ↓               ↓              ↓
    (raw data)    (merged+enriched)  (public)
```

**Per-source tables:** `scraper_maratonypolskie`, `scraper_datasport`, etc. — raw scraper output, upserted by `source_id`.

**Merge table:** `scraper_all` — cross-source deduped + merged with priority (dostartu > biegiwpolsce > timekeeper > elektronicznezapisy > datasport > maratonypolskie).

**Public table:** `calendar_events` — published rows with `status = 'pending'` or `'active'`. Admin approves via `/calendar-events` → "Do przeglądu" tab.

### Running the full pipeline

```bash
cd backend

# 1. Scrape raw data (6 sources → per-source tables)
node --env-file=../.env scripts/run-scrapers.js

# 2. Merge into scraper_all (cross-source dedup)
node --env-file=../.env scripts/run-merge.js --apply

# 2.5. Dedup scraper_all (same-source duplicates)
node --env-file=../.env scripts/run-dedup.js --apply

# 3. Geocode missing voivodeships/coordinates
node --env-file=../.env scripts/run-geocode.js --apply

# 4. Enrich flags (types, kids, distances from keywords)
node --env-file=../.env scripts/run-enrich-flags.js --apply

# 4.5. Normalize voivodeships and event types
node --env-file=../.env scripts/run-normalize.js --apply

# 5. PRIMARY ENRICHMENT — Python enricher (local Ollama LLM)
cd ../enricher && source .venv/bin/activate
docker compose up -d  # Start SearXNG
python -m enricher run

# 5.1. OPTIONAL — Claude CLI fallback (for fields enricher missed)
cd ../backend
node --env-file=../.env scripts/run-enrich-search.js --apply

# 5.5. Final dedup pass
node --env-file=../.env scripts/run-dedup.js --apply

# 6. Publish to calendar_events
node --env-file=../.env scripts/run-publish.js --apply

# 7. Generate static event pages manifest + OG images
node --env-file=../.env scripts/publish-event-pages.js --apply
```

### Python Enricher — PRIMARY enrichment tool

**Location:** `enricher/` directory

**Tech stack:**
- **Ollama** — local LLM (gemma3:27b) for field extraction
- **SearXNG** — Docker-based web search for URL discovery
- **Crawl4AI** — headless browser for crawling SPAs (dostartu.pl, etc.)
- **Docling** — PDF text extraction for regulamin documents

**What it enriches:**
- `registration_url` — LLM extracts from page content, fallback to SearXNG search
- `regulamin_url` — LLM extracts from page content or PDF links, fallback to SearXNG search
- `website` — official event site (SearXNG search + LLM validates it's not news/social/aggregator)
- `distances` — from regulamin PDFs (via Docling), registration pages, or website content (all crawled via Crawl4AI)
- `event_types` — `[trail, uliczny, nocny, ocr, nordic walking, ultra, charytatywny]` extracted from all content sources
- `price_from` / `price_to` — entry fees in PLN (prioritizes regulamin PDF over registration page, looks for "opłata startowa" tables with date tiers)
- `registration_deadline` — from regulamin or registration page (format: YYYY-MM-DD)
- `voivodeship` — only fills empty, never overwrites scraper's geocoded value
- `is_kids` — true if any distance ≤ 1 km or dedicated children's category exists

**Performance:** ~2 min/event (LLM inference on 32B model).

**Why it's the primary tool:** Cost-free (local model), comprehensive (crawls pages + PDFs), more reliable than API-based enrichment.

See [enricher/README.md](enricher/README.md) for detailed documentation.

### Admin tools for calendar management

- **Do przeglądu** — `/calendar-events` → approve/reject pending events
- **Duplikaty** — `/calendar-events` → find and merge duplicate entries
- **Manual event entry** — `/calendar-events/new` — add events found on Facebook or elsewhere
- **Calendar events API** — `GET/POST/PATCH/DELETE /api/calendar-events`

### Potential future scraper targets

- **extremalny.pl** — OCR/obstacle races (50-100 events)
- **przeszkodowo.pl** — OCR/obstacle races
- **biegigorskie.pl** — mountain/trail only, yearly HTML tables (easy)
- **zawodybiegowe.pl** — all types (200-500 events)
- **ligabiegowa.pl** — road running league

## Local LLM Enricher

Python-based enrichment pipeline in `enricher/`. Validates URLs, searches SearXNG, crawls pages with Crawl4AI, extracts PDFs with Docling, and uses Ollama (gemma3:27b) for field extraction. Only processes future events (date >= today).

### Running

```bash
cd enricher
source .venv/bin/activate
docker compose up -d          # SearXNG
python -m enricher run         # process all un-enriched future events
python -m enricher run --force              # re-process already-enriched events
python -m enricher run --limit 5 --dry-run  # test run
python -m enricher sync --since today --dry-run  # preview sync to calendar_events
python -m enricher sync --since today            # push to calendar_events
```

See `enricher/README.md` for full docs, all flags, merge rules, and architecture.

### Dependencies
- Ollama (native macOS, `gemma3:27b` — 128K context window, general-purpose extraction model)
- SearXNG (Docker via `enricher/docker-compose.yml`, port 8888)
- Crawl4AI + Docling (Python libs in `enricher/.venv/`)

### Key merge safety rules
- Never downgrades trail/ocr/charytatywny → uliczny (scraper keyword evidence preserved)
- Never nulls a working URL without a replacement candidate
- Never overwrites existing voivodeship (geocoding is more reliable than LLM)
- Rejects deadlines >1 year from event date (catches hallucinated years)
- Allows price 0 for free events, rejects price_from > price_to
- Keyword chunk extraction feeds focused price/deadline sections to LLM (not raw 6k char dumps)

## Data that persists across docker compose down

Named volume `pgdata` in docker-compose.yml. Never use anonymous volumes.
`docker compose down` is safe. `docker compose down -v` DELETES ALL DATA — warn user.

## UI Design Theme — "OVERDRIVE"

Dark, tactical theme for trail running timing. Acid yellow on near-black.

### Palette
- Background: `#0A0A10` (near-black)
- Surface: `#0C0C14` / `#12121E` / `#1A1A28`
- Borders: `#1C1C2A` / `#262638` / `#343450`
- Primary accent: `#BBDD00` (acid yellow), bright: `#D4FF00`, dim: `#778800`
- Text: `#B0AEC6` (body), `#DDDCEC` (bright/headings), `#8886A0` (muted)
- Red: `#EF4444`, Cyan: `#00BFEF`
- Forest green (logo leaves): `#2D5A27`

### Typography
- Headers: **Barlow Condensed** ExtraBold (font-display)
- Body: **Rajdhani** 500 (font-sans)
- Numbers/data: **IBM Plex Mono** (font-mono)

### Component rules
- Buttons: `rounded-none` — sharp edges, no pill shapes. Border-style (border+text, fills on hover)
- Badges: rectangular, thin border, color-coded per event type
- Cards: flat, thin border (`border-apex-border`), no shadow, yellow left-edge stripe on hover
- Inputs: `rounded-none`, `border-apex-border`, focus ring in yellow-dim
- Status colors: green=finished, cyan=active, yellow=dns/dnf, red=dsq

### Tailwind v4
Custom tokens defined via `@theme` directive in `app.css`. All use `apex-*` prefix (e.g., `bg-apex-surface`, `text-apex-yellow`).

### WCAG AA contrast
All color combos verified. Key ratios: text-bright on bg = 14.95:1, yellow on bg = 12.96:1, muted on bg = 5.75:1.

### Fonts (import in index.html)
```html
<link href="https://fonts.googleapis.com/css2?family=Barlow+Condensed:wght@700;800&family=Rajdhani:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;600&display=swap" rel="stylesheet">
```

## RFID Audit Queries

DB credentials: container `leszyrun-db-1`, user `leszyrun`, db `leszyrun`.

**Investigation flow:** start with query 5 to get EPC → query 1 to check R700 saw the tag → if rows exist, use query 2 to check antenna coverage.

**1. Did the R700 see this EPC at all?**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT received_at, antenna_port, rssi_cdbm, topic FROM gate_events WHERE epc = '<EPC>' ORDER BY received_at;"
```
Zero rows = hardware/RF issue. Rows present = detector logic issue.

**2. What did each antenna see for a participant?**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT antenna_port, COUNT(*) as pings, MIN(rssi_cdbm) as worst_rssi, MAX(rssi_cdbm) as best_rssi FROM gate_events WHERE epc = '<EPC>' GROUP BY antenna_port ORDER BY antenna_port;"
```

**3. All pings for a race — full timeline**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT ge.received_at, ge.epc, ge.antenna_port, ge.rssi_cdbm, p.first_name, p.last_name FROM gate_events ge LEFT JOIN participants p ON p.rfid_epc = ge.epc WHERE ge.race_run_id = '<RACE_RUN_ID>' ORDER BY ge.received_at;"
```

**4. Participants with no finish crossing**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT p.first_name, p.last_name, p.rfid_epc, r.status, r.start_time, r.finish_time FROM results r JOIN participants p ON p.id = r.participant_id WHERE r.race_run_id = '<RACE_RUN_ID>' AND r.finish_time IS NULL ORDER BY r.start_time;"
```

**5. Find EPC by participant name**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT first_name, last_name, rfid_epc FROM participants WHERE last_name ILIKE '%<NAME>%';"
```

**6. Find race_run_id for a recent race**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT rr.id, rr.started_at, rr.status, c.name as category FROM race_runs rr JOIN categories c ON c.id = rr.category_id ORDER BY rr.created_at DESC LIMIT 10;"
```

**7. Ping density per minute — spot RF blackout windows**
```bash
docker exec -it leszyrun-db-1 psql -U leszyrun -d leszyrun \
  -c "SELECT date_trunc('minute', received_at) as minute, COUNT(*) as pings FROM gate_events WHERE race_run_id = '<RACE_RUN_ID>' GROUP BY 1 ORDER BY 1;"
```

## Database write safety

**Before running any INSERT, UPDATE, or DELETE on any database (local Postgres or Supabase), you MUST:**

1. State exactly what will change — which table(s), which rows (with ID ranges or counts), which columns, what values
2. State the damage impact — is it reversible, what happens if it's wrong, does it affect synced data, other tables via cascades, production users
3. Wait for explicit user confirmation ("yes", "ok", "proceed", etc.) before executing

This applies to: `psql -c "DELETE..."`, `psql -c "UPDATE..."`, `psql -c "INSERT..."`, `mcp__supabase__execute_sql` with mutations, `mcp__supabase__apply_migration`, any ORM writes via scripts.

SELECT / read-only queries do NOT need confirmation.

Exception: if the user has just *explicitly* asked for the specific destructive action (e.g. "delete all race data for event X"), proceed without re-confirming, but still briefly state what you're about to do before doing it.

## Things to never do

- Do not use TypeScript (this is a JS project)
- Do not use Express (use Fastify)
- Do not use Prisma (use Drizzle)
- Do not use Socket.io (use `ws`)
- Do not use Next.js (use Vite)
- Do not dockerize Mosquitto (hardware constraint — R700 needs LAN access)
- Do not use `docker compose down -v` unless explicitly asked
- Do not pull data from Supabase into local DB (exception: `checkins` and `checkin_documents` via reverse sync)
- Do not add TypeScript type annotations or `.ts` files
- Do not use peak RSSI for signal-strength bars — always use live (most recent) reading with decay. See ARCHITECTURE.md → "RSSI display rule — live signal, not peak"
- Do not create local copies of `estimatePositions()` — always import from `@leszyrun/ui` (shared package in `packages/ui/src/lib/positionEstimation.js`). A stale local copy caused a live-race bug where podium ordering ignored checkpoint timestamps. If you think the shared function needs changes, stop and ask the user first — the sorting tiers (finish time → checkpoint index → observation time → start time) are load-bearing for live race display.
- Do not re-run `estimatePositions()` inside `CategoryCard` when `resultsProp` is provided — the caller already enriched results with checkpoint observations. Re-estimating with empty observations discards checkpoint data and breaks podium ordering during live races. See the comment in `frontend/src/pages/PodiumPage.jsx`.
- Do not filter race runs to only `'active'` status in podium or public result views — always include `'finished'` too. Filtering only active causes the podium/results to go blank the moment a race is stopped. The podium and public views must keep showing final results after the race ends.
- Do not add `Co-Authored-By:` trailers to git commits. Never include Claude authorship in commit messages.
- Do not permanently delete calendar events without asking the user for confirmation first. Prefer rejecting (setting status to `rejected`) over deleting — rejected events prevent the scraper from re-adding the same junk.
- Do not DELETE rows from any Supabase table unless the user explicitly says "delete from [table name]". When asked to "remove" an event, ask which table(s) — never assume. Scraper source tables (`scraper_*`) are raw data and should almost never be touched directly.

---
> Source: [derberg/leszy.run](https://github.com/derberg/leszy.run) — distributed by [TomeVault](https://tomevault.io/claim/derberg).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
