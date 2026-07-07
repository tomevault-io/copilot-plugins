## algovictory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Backend (Django)
```bash
# Python deps managed with uv (pyproject.toml + uv.lock)
uv sync                    # install/sync deps
source .venv/bin/activate  # activate venv before running manage.py

python manage.py runserver

# Migrations
python manage.py makemigrations
python manage.py migrate

# Django shell
python manage.py shell

# Linting
ruff check .
```

### Frontend (Vite + React)
```bash
cd frontend
npm run dev       # dev server on http://127.0.0.1:5173
npm run build     # tsc + vite build → dist/
npm run lint      # eslint
```

### Stripe webhook testing (local)
```bash
stripe listen --forward-to 127.0.0.1:8000/payments/stripe-webhook/
stripe trigger checkout.session.completed
```

### Production (Docker)
```bash
docker-compose up --build   # backend on :8000 (internal), frontend on :8280
# VITE_API_URL must be set in environment — passed as build arg to frontend image
```

### Test scripts (ad hoc, not a test suite)
```bash
# Located in testScripts/ — run individually, e.g.:
python testScripts/endpointTest.py
```

There are no automated test suites. `testScripts/` contains one-off Python scripts for manual endpoint and flow verification.

## Architecture

### Django apps

| App | Responsibility |
|---|---|
| `api` | Custom `User` model (extends `AbstractUser`), auth views, allauth adapter |
| `sports` | MMA data models, sportsdata.io API jobs, AI analysis via Groq |
| `payments` | Stripe checkout session + webhook handler |
| `divinatio` | Django project config; `apps.py` starts APScheduler on server boot |

### URL routing
- `api/token/` — JWT obtain/refresh (simplejwt)
- `api/user/register/` — user registration
- `api/` → `api.urls` — user-info, social token exchange, password reset
- `sports/` → `sports.urls` — event list, fight card, AI analysis
- `payments/` → `payments.urls` — Stripe pay + webhook
- `accounts/` → allauth (Google/Twitter OAuth)

**Important:** the axios `baseURL` in the frontend defaults to `http://127.0.0.1:8000` (not `/api`). `sports/` and `payments/` endpoints are at the backend root. Calls must include the full prefix (e.g. `api.get('/sports/fight-card/')`).

### Auth flow
Two auth paths converge on JWT stored in `localStorage`:
1. **Email/password** — POST to `api/token/` → store `access`/`refresh` tokens
2. **Social (Google/Twitter)** — allauth session login → Django redirects to `api/social-token/` → exchanges session for JWT → redirects to frontend `/home?access=...&refresh=...`

`AuthContext` (`frontend/src/components/AuthContext.tsx`) is the central auth state. It checks token expiry on mount, auto-refreshes, and fetches `api/user-info/` to populate `username`, `avatar`, and `isPremium`. Wrap protected UI in `<ProtectedRoute>` (requires auth) or `<LogedInLock>` (blocks already-logged-in users).

### Sports data ingestion (APScheduler)
`divinatio/apps.py` → `runapscheduler.py` starts APScheduler when the Django dev server's main process launches (`RUN_MAIN=true`). Jobs fire **immediately on startup** (except `send_card_to_groq`, which is commented out from startup), then repeat on their interval:

| Job | Interval | What it does |
|---|---|---|
| `fetch_mma_schedules` | 20 days | Pulls UFC schedule from sportsdata.io, upserts `UpcomingEventsModel` |
| `send_card_to_groq` | 20 days | Sends fight card to 4 Groq models, stores results in `AisResponseModel` |
| `get_fighter_stats` | 15 days | Fetches fighter stats, populates `FighterModel` + `FightModel` + `FightFighterModel` |
| `set_fighter_image` | 15 days | Fills missing `imageURL` on `FighterModel` via octagon-api.com |
| `schedule_archive` | 1 day | Schedules one-shot `archive_event` to run the day after the next event |
| `expire_memberships` | 1 day | Sets `User.premium = False` when `payment_expires` has passed |

Jobs are stored in the DB via `DjangoJobStore` and visible in the Django admin.

### Data model relationships (sports app)
```
UpcomingEventsModel (eventId unique) ──archived──▶ PastEventsModel
    ├── FightModel (FK → UpcomingEventsModel.eventId)
    │       └── FightFighterModel (FK → FightModel, FK → FighterModel)
    │               └── FighterModel (fighter_id PK)
    └── AisResponseModel (FK → UpcomingEventsModel.eventId, JSONField: chatter)
```

`FightFighterModel` is the M2M join table; it also holds `moneyline` and pre-fight record (wins/losses/draws). `FightFighterSerializer` only exposes the nested `fighter` object — moneyline and pre-fight record are not in the API response.

`archive_event()` copies the first `UpcomingEventsModel` row to `PastEventsModel` using a `__class__` reassignment trick, then deletes it.

### AI analysis pipeline
`sports/ais.py` → `send_card_to_groq()` sends the current fight card to four Groq models (Qwen 3-32B, Llama 3.1-8B, Llama 3.3-70B, Llama 4 Scout) and stores the combined JSON in `AisResponseModel.chatter`. The endpoint `sports/ai/` (`AiAnalysisView`) is gated by the custom `PremiumUser` permission class. The frontend displays results at `/test` via `AiAnalysisView` component using a consensus vote panel + per-model fighter cards.

### Frontend structure
- `src/api.ts` — configured axios instance; reads `VITE_API_URL` from env, falls back to `http://127.0.0.1:8000`
- `src/interfaces.ts` — all shared TypeScript interfaces mirroring backend serializers
- `src/App.tsx` — route definitions; lazy-loads heavy pages (`PasswordReset`, `PaymentResponse`, `Settings`, `PrivacyPolicy`, `TermsOfService`)
- MUI + Tailwind CSS v4 are both active; MUI theme is in `src/styles/theme`

### Environment variables

**Backend (`.env` in repo root):**
| Variable | Purpose |
|---|---|
| `production` | If set, enables prod mode — ports/URLs read from env, CORS origins read from env |
| `DJANGO_SECRET_KEY` | Django secret key |
| `SENDGRID_API_KEY` | Transactional email |
| `STRIPE_SECRET` | Stripe API key |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signature verification |
| `SPORTS_API_KEY` | sportsdata.io MMA API |
| `GROQ_API_KEY` | Groq API key (used by `sports/ais.py` via the Groq client) |
| `DB_PATH` | SQLite file path (defaults to `db.sqlite3` in repo root) |
| `ALLOWED_HOSTS` | Comma-separated hosts (prod only) |
| `CORS_ALLOWED_ORIGINS` | Comma-separated origins (prod only) |
| `FRONTEND_URL` | Frontend base URL for JWT redirect (prod only) |

**Frontend (`frontend/.env`):**
| Variable | Purpose |
|---|---|
| `VITE_API_URL` | Backend base URL (default: `http://127.0.0.1:8000`) |
| `VITE_PROD_WIP` | Shows `ConstructionBanner` when true |

### Premium / payments
`payments/views.py` creates Stripe checkout sessions and handles `checkout.session.completed` webhooks. On success, `User.premium = True` and `User.payment_date` + `User.payment_expires` (1 month out) are recorded. The daily `expire_memberships` job reverts premium when the expiry passes. `AuthContext` exposes `isPremium` from `/api/user-info/`.

### Shared dev config (`api/dev.py`)
`api/dev.py` centralises port/URL constants and provides `get_logger()`. In dev mode (no `production` env var), all ports and URLs are hardcoded defaults. Import from here rather than repeating `os.getenv` calls in view files.

---
> Source: [AquilaIgnis/AlgoVictory](https://github.com/AquilaIgnis/AlgoVictory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
