## kiraimobiliare

> Romanian rental-listings aggregator: Python scraping worker + FastAPI + React SPA.

# RentScalper

Romanian rental-listings aggregator: Python scraping worker + FastAPI + React SPA.
Target cities: Cluj-Napoca, Oradea, Timisoara, Iasi, Targu Mures, Bucuresti.

## Layout
- `backend/app/scraping/sites/` — one adapter per source site (contract: `scraping/base.py`; reference impl: `storia.py`). Register new adapters in `sites/__init__.py`.
- `backend/app/scraping/extractors/` — pure-regex Romanian text extraction (parking taxonomy, heating, price, rooms, street). All matching on diacritics-folded text (`core/textutil.fold`).
- `backend/app/scraping/pipeline.py` — RawListing -> extract -> geocode -> upsert (idempotent by URL).
- `backend/app/data/cities/*.json` — source of truth for zones/nearby-towns/site slugs. `verified: false` means centroid not yet confirmed via Nominatim (`backend/tools/geocode_zones.py`).
- `backend/app/services/` — geo (Nominatim + DB cache, budget-capped), matching (parking<->rent rebuild per scrape).
- `backend/app/api/routes/auth.py` + `billing.py` — accounts (email+bcrypt, Google ID token, JWT httpOnly cookie) and Stripe subscription (Checkout/Portal/sync/webhook).
- `frontend/` — Vite + React TS + Tailwind v4 (class-based dark mode); all filter state in URL searchParams; UI text Romanian.

## Commands (backend dir, venv `.venv`)
- Tests: `.venv\Scripts\python.exe -m pytest tests -q`
- API: `.venv\Scripts\python.exe -m uvicorn app.main:app --reload --port 8000`
- One-shot scrape: `.venv\Scripts\python.exe -m app.worker.scheduler --once --city cluj-napoca [--site storia] [--max-pages 2]`
- Scheduler daemon: `.venv\Scripts\python.exe -m app.worker.scheduler`
- Frontend: `npm run dev` in `frontend/` (proxies /api -> :8000)

## Conventions / invariants
- Datetimes: naive UTC everywhere (`models.utcnow()`); SQLite-safe comparisons.
- Listing "active" = `last_seen_at` within `RS_LISTING_ACTIVE_DAYS` (no delete jobs).
- Card click goes to the ORIGINAL listing URL; images go through `/api/img?u=` proxy (host whitelist = adapters' `image_hosts`).
- Parking taxonomy: included / likely_included / area_possible / none / unknown + separate ParkingMatch rows for rentable spots nearby. `is_approx` when either side geocoded at zone/city precision.
- Scrapers must stay polite: PoliteSession (per-host delay), page caps, budget-capped detail fetches; never retry non-200. Identifiable UA ("KiraBot", points to /despre); only scrape robots.txt-allowed listing pages (verified: storia/olx/imobiliare/publi24 allow the result+offer paths we hit).
- GDPR data minimisation: `core/textutil.redact_personal` strips phone numbers + emails from stored title/description (applied in `pipeline.apply_extractions` + `upsert_parking`). Never store contact details — users get them on the original ad we link to.
- Legal: in-app /despre (good-faith aggregation + source opt-out), /termeni has an IP + anti-scraping clause, /confidentialitate, /cookies. Product name "Kira" © in Terms.
- Settings via `.env` at repo root, prefix `RS_` (see `.env.example`).
- Freemium paywall: free users get first `RS_FREE_LISTING_LIMIT` listings, but `total` in `/api/listings` is ALWAYS the real count (product requirement). Gating lives ONLY server-side in `listings.py`; never trust the client.
- Billing: ONE-TIME 15 RON Stripe payment = 30 days access (`mode="payment"`, NOT a subscription, nothing auto-renews, no Billing Portal). `grant_access` stacks 30d (extends from current end if still active). `has_access()` is purely date-based (`sub_period_end > now`). Each paid Checkout session credited once via `last_payment_session`. Checkout has an idempotency key. First launch month is free (communicated in UI/Terms). Redirect params: `?plata=succes|anulat`.
- Auth endpoints are rate-limited per IP (`core/ratelimit.py`, in-memory — swap for Redis if multi-worker).
- Account deletion (`/auth/delete-account`) must cancel live Stripe subs first (GDPR erasure without zombie charges).
- Deploy: single Hetzner VPS (Caddy + uvicorn + worker via systemd) — see `docs/DEPLOY.md`. Prod checklist: RS_SECRET_KEY (32+ chars), RS_COOKIE_SECURE=true, RS_ENABLE_ADMIN_ENDPOINTS=false.
- Monitoring: `/api/admin` HTML dashboard (token-gated, server-rendered) + `/api/admin/stats` JSON (`_build_stats`): health verdict, per-source freshness ok/stale, last scrape runs, 24h errors, coverage %, users/subscribers. `services/alerts.py` emails (SMTP optional, deduped 6h) when a scrape errors or a source returns 0 — called from `worker/jobs.run_scrape_city`.

## Task delegation (user preference)
Use cheaper-model agents (sonnet/haiku) for: new site adapters (give them `storia.py` + an old-script reference), city data files, frontend components, docs. Keep for the main model: extractor semantics, pipeline/matching changes, API contract changes.

---
> Source: [Cezart3/KiraImobiliare](https://github.com/Cezart3/KiraImobiliare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
