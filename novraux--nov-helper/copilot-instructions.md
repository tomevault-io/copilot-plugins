## nov-helper

> Novraux is a **Print-on-Demand (POD) automation platform** for trend-driven product research, SEO optimization, and multi-store management. It integrates AI pipelines (Groq, Claude, OpenAI, Google AI) with Shopify, Printful, and Etsy to automate the design-to-sale workflow.

# CLAUDE.md — Novraux Codebase Guide

Novraux is a **Print-on-Demand (POD) automation platform** for trend-driven product research, SEO optimization, and multi-store management. It integrates AI pipelines (Groq, Claude, OpenAI, Google AI) with Shopify, Printful, and Etsy to automate the design-to-sale workflow.

---

## Architecture Overview

```
nov_helper/
├── backend/          # FastAPI Python backend (port 8000)
├── frontend/         # React + TypeScript + Vite dashboard (port 5173)
├── n8n/workflows/    # Automation workflow templates (n8n on port 5678)
├── docker-compose.yml
└── .env              # All secrets (never commit)
```

Three Docker services run together:
- **`novraux-postgres`** — PostgreSQL 16 database (port 5432)
- **`novraux-backend`** — FastAPI application with live reload
- **`novraux-n8n`** — n8n workflow automation engine

---

## Development Commands

### Start the full stack

```bash
docker compose up -d          # Start all containers (detached)
docker compose ps             # Check container health
docker compose logs -f backend  # Stream backend logs
docker compose down           # Stop all containers
```

### Frontend (React dev server)

```bash
cd frontend
npm install                   # Install dependencies
npm run dev                   # Dev server → http://localhost:5173
npm run build                 # TypeScript check + Vite production build
npm run lint                  # ESLint (0 max-warnings — must be clean)
npm run preview               # Preview production build locally
```

### Backend (outside Docker, for quick testing)

```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### Useful API calls

```bash
curl http://localhost:8000/health             # Health check → {"status":"ok"}
curl -X POST http://localhost:8000/trends/scrape  # Trigger manual scrape
curl http://localhost:8000/trends             # List all trends
```

---

## Environment Variables

All secrets live in `.env` at the project root (loaded by `docker-compose.yml` and `backend/config.py`).

| Variable | Required | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | Yes | DALL-E mockup generation |
| `AI_API_KEY` | Yes | Groq — fast/cheap trend scoring |
| `AI_API_BASE_URL` | Yes | `https://api.groq.com/openai/v1` |
| `GOOGLE_AI_KEY` | Yes | Google Gemini multimodal analysis |
| `ANTHROPIC_API_KEY` | Pending | Claude deep analysis for top trends |
| `SHOPIFY_STORE_URL` | Yes | Shopify Admin API store URL |
| `SHOPIFY_ACCESS_TOKEN` | Yes | Shopify Admin API access token |
| `PRINTFUL_API_KEY` | Pending | Printful mockup + fulfillment |
| `DATABASE_URL` | Auto-set | Overridden by Docker Compose |

Config is managed via `backend/config.py` using `pydantic_settings.BaseSettings`. Env file path resolves to `../.env` relative to the backend directory.

---

## Backend Structure

```
backend/
├── main.py              # FastAPI app entry point, CORS, startup hooks
├── config.py            # Settings via pydantic_settings (reads .env)
├── requirements.txt     # Python dependencies
├── Dockerfile           # Python 3.11 slim image
├── db/
│   ├── database.py      # SQLAlchemy engine + session factory
│   └── models.py        # ORM models (Trend table)
├── routers/
│   ├── trends.py        # GET/POST /trends — trend feed and scraping
│   ├── shopify.py       # GET/POST /shopify — product fetch, SEO push
│   ├── seo.py           # POST /seo — SEO generation endpoints
│   ├── orders.py        # GET /orders — order management
│   └── research.py      # POST /research — competitor/niche analysis
└── services/
    ├── ai/
    │   ├── groq_client.py      # Groq LLM — fast trend scoring
    │   ├── claude_client.py    # Anthropic Claude — deep analysis (7+ score)
    │   ├── seo_generator.py    # SEO title/description/tags generation
    │   └── gap_analyzer.py     # Market gap analysis
    ├── scrapers/
    │   ├── google_trends.py    # pytrends — Google Trends data
    │   ├── tiktok_trends.py    # TikTok hashtag scraping
    │   ├── pinterest_trends.py # Pinterest search trend scraping
    │   ├── redbubble_trends.py # Redbubble popular tags (competitor data)
    │   └── etsy_insights.py    # Etsy keyword insights
    ├── helpers/
    │   ├── blacklist.py        # Filters brand names and offensive terms
    │   └── temporal_detector.py # Seasonal/temporal trend tagging
    ├── shopify.py       # Shopify Admin API client
    ├── printful.py      # Printful API client
    └── orders_sync.py   # Sync orders from Shopify/Etsy
```

### Key conventions

- **Routers** use `APIRouter` with a `prefix` and `tags` defined at the top. Each router file is self-contained.
- **Services** are pure functions or async functions — no FastAPI dependencies injected.
- **DB sessions** are injected via `Depends(get_db)` in router handlers; never instantiate sessions manually.
- **Tables are auto-created** on startup via `create_tables()` called in `main.py`'s `on_startup` event. There is no Alembic migration workflow currently — schema changes require a container restart or manual `DROP TABLE`.
- **AI gating**: Groq scores every trend cheaply first. Claude deep analysis is only triggered for trends scoring **≥ 7.0** to minimize API costs.
- **Caching**: Trends have a **48-hour scrape cache** — the scraper skips re-fetching keywords seen within that window. Cost per trend is tracked in the `Trend` table fields (`scoring_cost`, `analysis_cost`, `total_api_cost`).

### Database model: `Trend`

Core fields on the `Trend` model (in `db/models.py`):

| Field | Type | Description |
|---|---|---|
| `keyword` | String | Trend keyword/phrase |
| `source` | String | Scraper source (google, tiktok, pinterest, redbubble) |
| `score_groq` | Float | Groq viability score (0–10) |
| `pod_viability` | Float | POD product viability sub-score |
| `competition_level` | String | low / medium / high |
| `ip_safe` | Boolean | Passes IP/trademark check |
| `deep_analysis` | Text | Claude-generated narrative analysis |
| `temporal_tags` | JSON | Seasonal tags (valentine, q4, etc.) |
| `avg_interest` | Int | Google Trends interest (0–100) |
| `trend_velocity` | String | rising / stable / declining |
| `total_api_cost` | Float | Cumulative API spend for this trend |

---

## Frontend Structure

```
frontend/
├── src/
│   ├── App.tsx          # Root component — 4-tab sidebar navigation
│   ├── main.tsx         # React entry point
│   ├── api.ts           # API client (all fetch calls to localhost:8000)
│   ├── types.ts         # TypeScript interfaces matching backend Pydantic models
│   ├── pages/
│   │   ├── TrendFeed.tsx       # Trend list with filters and live scraper
│   │   ├── ShopifySEO.tsx      # Product list + SEO generation + bulk push
│   │   ├── NicheExplorer.tsx   # Research and competitive intelligence
│   │   └── Orders.tsx          # Order list and revenue tracking
│   └── components/
│       ├── TrendCard.tsx        # Individual trend display card
│       ├── ScoreBadge.tsx       # Color-coded score badge (0–10 scale)
│       └── StatsGrid.tsx        # Metrics summary grid
├── package.json
├── tsconfig.json
├── vite.config.ts
└── index.html
```

### Key conventions

- **`api.ts`** is the single API client. All backend calls go through named functions here (e.g. `getTrends()`, `scrapeNow()`, `generateSEO()`). Base URL is `http://localhost:8000`.
- **`types.ts`** mirrors the backend's Pydantic response models. Keep these in sync when adding/changing backend fields.
- ESLint is configured with `--max-warnings 0` — the build will fail on any warning. Fix warnings before committing.
- TypeScript strict mode is enabled. Avoid `any` types.
- No CSS framework is currently used — styling is inline or via plain CSS classes.

---

## AI Pipeline Design

The platform uses a **two-tier AI strategy** to balance speed and cost:

### Tier 1 — Groq (fast, cheap)
- Model: `llama-3.1-8b-instant`
- Used for: every trend score (POD viability, competition level, IP safety check)
- Output: structured JSON with `score`, `pod_viability`, `competition_level`, `ip_safe`, `product_suggestions`

### Tier 2 — Claude (quality, gated)
- Model: configured via `ANTHROPIC_API_KEY`
- Triggered only when `score_groq >= 7.0`
- Used for: deep narrative analysis, design brief, target audience profiling
- Output: freeform text stored in `deep_analysis`

### SEO Generation
- `services/ai/seo_generator.py` generates Etsy/Shopify-optimized titles, descriptions, and 13-tag sets
- Groq is the default model for SEO to keep costs low; Claude can be used for refinement

---

## Data Sources

| Scraper | Status | Notes |
|---|---|---|
| Google Trends (`pytrends`) | Active | 48h cache, interest score 0–100 |
| Redbubble popular tags | Active | Competitor validation |
| TikTok hashtags | Scaffolded | API key not yet configured |
| Pinterest trends | Scaffolded | API approval pending |
| Etsy keyword insights | Scaffolded | API approval pending |

---

## n8n Automation (port 5678)

Default credentials: `admin` / `novraux_n8n`

Planned workflows (templates in `n8n/workflows/`):
- **Scheduled scraper** — triggers `POST /trends/scrape` every 6 hours
- **Daily digest** — sends top 5 trends to Telegram/email each morning
- **Order sync** — polls Shopify/Etsy every 30 minutes

---

## Project Status (as of Feb 2026)

| Phase | Status | Notes |
|---|---|---|
| Phase 0 — Foundation | In Progress | Etsy shop live, Printful connected; first listing not yet published |
| Phase 1 — Trend Engine | Mostly Complete | Google Trends + Groq scoring + Claude analysis + dashboard working; n8n automation TODO |
| Phase 2 — Shopify SEO | Complete | Product fetch, AI SEO generation, bulk push to Shopify all functional |
| Phase 3 — Management Dashboard | Not Started | Order sync, unified revenue view, Telegram alerts |

---

## Common Pitfalls

- **Do not rename or drop `Trend` table columns** without considering that there are no Alembic migrations. Schema changes require either a data migration script or wiping the volume.
- **48-hour scrape cache** means `POST /trends/scrape` is a no-op if the trend was scraped recently. To force a re-scrape during development, clear the `last_scraped_at` fields or truncate the table.
- **Groq model name matters** — the API will error on deprecated model IDs. Current working model: `llama-3.1-8b-instant`.
- **CORS origins** in `main.py` are hardcoded to `localhost:5173` and `localhost:3000`. Add your origin if deploying to a different host.
- **`.env` is gitignored** — never commit it. New contributors need to copy `.env.example` (if one is created) or set variables manually.
- **ESLint `--max-warnings 0`** means `npm run lint` fails on any warning, not just errors. The CI/CD pipeline (if added) will fail too.

---

## Adding a New Feature — Checklist

1. **Backend endpoint**: Add a new file in `routers/` or extend an existing one. Register the router in `main.py`.
2. **Service logic**: Add business logic to `services/`. Keep routers thin.
3. **DB change**: Add/modify the model in `db/models.py`. Restart the backend container to apply (no migration workflow yet).
4. **Frontend type**: Update `src/types.ts` to match new response shape.
5. **API client**: Add a new function in `src/api.ts`.
6. **UI**: Add or update a page in `src/pages/` and wire it into `App.tsx` if it's a new tab.
7. **Lint**: Run `npm run lint` — fix all warnings before committing.
8. **Docker**: Run `docker compose up -d --build` if you changed `requirements.txt` or `Dockerfile`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/novraux)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/novraux)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
