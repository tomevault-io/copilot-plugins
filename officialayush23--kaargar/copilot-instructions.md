## kaargar

> Kaargar is a dual-mode (Instant + Discovery) hyperlocal services marketplace for Pune, built on

# KAARGAR — Claude Code Build Context

Kaargar is a dual-mode (Instant + Discovery) hyperlocal services marketplace for Pune, built on
FastAPI + Supabase Postgres + React/Vite. The system is **already built and running** — this file
is a quick orientation and a set of hard rules for anyone (human or Claude) editing this repo.
For how the system actually works today (auth, matching engine, booking flows, job lifecycle,
penalties, admin config, gaps), see **`SYSTEM_OVERVIEW.md`** in this same directory. Read that
before making non-trivial changes — do not re-derive behavior from old specs.

Two older docs, `CLAUDE_LEGACY.md` and `KAARGAR_SYSTEM_DESIGN_v2_LEGACY.md`, are kept for history
only. They describe an OTP-based auth flow and a Cloudinary storage backend that were never built
this way, and other since-changed decisions. **Do not use them as a source of truth** — they are
archived, not current.

---

## CRITICAL RULES (unchanged, still enforced)

1. Frontend: **React + Vite + JSX only** — no TypeScript, no `.tsx` files.
2. Backend: **all SQLAlchemy models live in `BACKEND/models.py`**, **all Pydantic schemas live in
   `BACKEND/schemas.py`** — no exceptions, no `models/` or `schemas/` packages.
3. Media uploads: **Supabase Storage only.** No Cloudinary, no S3. There are now **six** buckets
   in active use (see `BACKEND/services/storage.py`), not the original two — see below.
4. Auth: **Supabase email+password**, validated on the backend as a Supabase-issued JWT. There is
   **no OTP send/verify flow** and the backend **does not issue its own JWT** — see
   `SYSTEM_OVERVIEW.md` for the full picture and why `APP_SECRET_KEY`/`jwt_secret_key` in
   `config.py` is present but currently unused.
5. Real-time: **Supabase Realtime** `postgres_changes` subscriptions on the frontend (no polling
   except the backend's own 500ms dispatch-acceptance poll during instant matching).
6. Background jobs: **APScheduler inside the FastAPI process** (`BACKEND/main.py` lifespan) — no
   Celery, no external job queue.
7. Redis: used for the **instant-dispatch lock** (`services/matching.py`), a **translation cache**
   (`services/translation.py`), and a lookup in `routers/workers.py`. It fails open (skips the
   lock) if unreachable — it is not a hard dependency. It is *not* currently used for OTP-send
   rate limiting (there is no OTP flow to rate-limit).
8. Payments: **Razorpay mandatory** — no cash option. Commission/GST is computed server-side in
   `services/matching.calc_commission` and deducted from the worker's payout (not added as a
   customer surcharge). Rates and thresholds are DB-config-driven via `platform_config`, with
   hardcoded fallback defaults — see `SYSTEM_OVERVIEW.md`.
9. Notifications: **SMTP-style email (via Resend) + Supabase Realtime in-app** — no push/FCM.
10. Discovery bookings always pin a specific worker chosen by the customer. There is **no**
    "auto-match" / "let the system find someone" path for Discovery — that was deliberately
    removed. Only Instant jobs use the expanding-radius matching engine.

---

## REPOSITORY STRUCTURE (actual, verified)

```
Kaargar/
├── CLAUDE.md                        ← this file
├── SYSTEM_OVERVIEW.md               ← comprehensive current-state doc, read this next
├── CLAUDE_LEGACY.md                 ← archived, stale, do not use as source of truth
├── KAARGAR_SYSTEM_DESIGN_v2_LEGACY.md ← archived, stale, do not use as source of truth
├── README.md, TODO.md, IMPLEMENTATION_AUDIT.md, JOB_COMPLETION_FLOW_PLAN.md,
│   KAARGAR_BACKEND_UPDATE.md, Kaargar_UI_CrossCheck.md   ← various working/progress notes,
│                                                            not guaranteed current — verify
│                                                            against code before relying on them
├── DEPLOYMENT_TODO.md               ← actively-maintained checklist: Render Celery worker/beat
│                                        deploy steps, local Docker dev setup, mobile local-vs-prod
│                                        API URL setup, remaining roadmap (OTP/FCM/promo/restructure)
├── 001_initial_schema.sql … 017_worker_areas.sql  ← migrations, applied in order (013 has two
│   files — 013_backfill_instant_availability.sql and 013_service_catalog.sql — check both)
├── seed_data.sql
├── BACKEND/
│   ├── main.py                      ← FastAPI app, CORS, router mounting, APScheduler lifespan
│   ├── config.py                    ← pydantic-settings Settings (see env vars below)
│   ├── database.py
│   ├── models.py                    ← ALL SQLAlchemy models (43+, incl. DeviceToken/PromoCampaign/
│   │                                   PromoCampaignDelivery/ContentTranslation — added once
│   │                                   models.py was reconciled against the live DB)
│   ├── schemas.py                   ← ALL Pydantic schemas (1250+ lines)
│   ├── dependencies.py              ← get_current_user / require_worker / require_admin
│   ├── alembic.ini, alembic/{env.py, script.py.mako, versions/baseline_20260803.py}
│   │                                   ← real Alembic project (was in requirements.txt but never
│   │                                   wired up before). Live DB needs `alembic stamp
│   │                                   baseline_20260803` run once — see that file's docstring.
│   │                                   New schema changes should go through
│   │                                   `alembic revision --autogenerate` from here on, not hand-run SQL.
│   ├── Dockerfile, docker-compose.yml, .dockerignore  ← LOCAL DEV ONLY (redis + api + celery
│   │                                   worker + celery beat in one `docker-compose up`). Render
│   │                                   deploy is native Python, not Docker — see DEPLOYMENT_TODO.md.
│   ├── BACKEND_SPEC.md              ← another stale doc living inside BACKEND/ — same caveat
│   │                                   as the legacy docs, do not treat as current
│   ├── routers/
│   │   ├── auth.py                  ← /auth/provision, /auth/me, /auth/logout (no OTP here)
│   │   ├── categories.py, workers.py, jobs.py, upload.py, search.py, chat.py,
│   │   ├── payments.py, reviews.py, notifications.py, support.py, admin.py,
│   │   ├── users.py, geocode.py, addresses.py
│   ├── services/
│   │   ├── matching.py              ← instant dispatch engine + calc_commission
│   │   ├── notifications.py, storage.py, config.py, crypto.py, penalties.py, scheduling.py,
│   │   │   translation.py
│   ├── tasks/
│   │   ├── escrow_release.py, decay_scores.py, slot_rollover.py  ← all started via APScheduler
│   │   │   in main.py's lifespan
│   ├── templates/email/otp.html     ← legacy filename; used by services/notifications.py's
│   │                                   email-sending helper, not an OTP-flow endpoint
│   └── requirements.txt
└── FRONTEND/
    ├── index.html, vite.config.js, package.json, tailwind.config.js, postcss.config.js
    ├── FRONTEND_UI_SPEC.md, FRONTEND_UPDATE.md   ← working notes, verify against code
    └── src/
        ├── main.jsx, App.jsx, globals.css
        ├── lib/          api.js, supabase.js, utils.js, phoneCipher.js, theme.js
        ├── stores/       auth.js, app.js
        ├── i18n/         index.js
        ├── hooks/        useCategories, useJobs, useWorker, useNotifications, useAddresses,
        │                 useGeoLocation, useGeocoding, useLocationPublisher, useRazorpay
        ├── components/
        │   ├── ui/       shadcn-style JSX components (button, card, dialog, tabs, …)
        │   ├── glass/    Background, GlassButton/Card/Input/Modal/Navbar/Select, ThemeToggle
        │   ├── kaargar/  ModeToggle, CategoryGrid, WorkerCard, JobStatusTimeline,
        │   │             NotificationDrawer, SearchBar, MediaUpload, ActiveJobBar,
        │   │             JobCompletionFlow, JobTrackingMap, MapLocationPicker, PuneMap, …
        │   └── layout/   AppLayout, WorkerLayout
        └── pages/
            ├── auth/LoginPage.jsx
            ├── home/HomePage.jsx
            ├── job/NewJobPage, SearchingPage, ActiveJobPage, JobApprovalPage, JobDetailPage,
            │        ReviewPage
            ├── discovery/DiscoveryPage, BookDiscoveryPage, WorkerProfilePage
            ├── bookings/BookingsPage
            ├── chat/ChatPage
            ├── profile/ProfilePage, SupportPage
            ├── onboarding/WorkerOnboardPage
            ├── worker/WorkerDashboard, IncomingJobModal, WorkerAnalytics, WorkerServices,
            │          WorkerMedia, WorkerProfile, WorkerSchedule, WorkerPackages,
            │          WorkerOffers, WorkerSupport
            └── admin/AdminDashboard, AdminLogin, AdminLayout, AdminUsers, AdminWorkers,
                       AdminJobs, AdminPayouts, AdminSupport, AdminCategories, AdminConfig
```

---

## ENVIRONMENT VARIABLES

These are read straight from `BACKEND/config.py`'s `Settings` class and `FRONTEND/src`'s
`import.meta.env.VITE_*` usages — verified against actual code, not assumed. Both
`.env.example` files have been updated to match (see below); if you add a new setting, add it to
`config.py` **and** to the matching `.env.example`.

### Backend (`BACKEND/.env`, loaded via `config.py`)

| Var (as read from environment) | Settings field | Default if unset | Notes |
|---|---|---|---|
| `ENVIRONMENT` | `app_env` | `"development"` | |
| `FRONTEND_URL` | `frontend_url` | `https://kaargar1.vercel.app` | normalized to have a scheme; used for CORS allow-list |
| `FASTAPI_HOST` | `fastapi_host` | `"0.0.0.0"` | |
| `FASTAPI_PORT` | `fastapi_port` | `8000` | |
| `DATABASE_URL` | `database_url` | **required** | `postgresql://`/`postgres://` auto-rewritten to `postgresql+asyncpg://`. Supavisor pooled (transaction mode, port 6543) — what the running app uses. |
| `DATABASE_URL_DIRECT` | `database_url_direct` | `""`, falls back to `database_url` | Unpooled direct connection (port 5432). Used by `BACKEND/alembic/env.py` only — migrations run session-level DDL/advisory-lock operations that don't reliably work through a transaction-mode pooler. |
| `SUPABASE_URL` | `supabase_url` | **required** | |
| `SUPABASE_ANON_KEY` | `supabase_anon_key` | `""` | |
| `SUPABASE_SERVICE_ROLE_KEY` | `supabase_service_role_key` | **required** | used by `services/storage.py` to write to Storage buckets |
| `SUPABASE_JWT_SECRET` | `supabase_jwt_secret` | `""` | **this is the real auth secret** — validates the Supabase-issued JWT in `dependencies.py`/`routers/auth.py`. Required for auth to work at all. |
| `APP_SECRET_KEY` | `jwt_secret_key` | **required** | defined for a custom-JWT flow that was never implemented; not currently used anywhere for signing/verifying tokens. Kept for forward-compatibility if auth is ever moved off Supabase. |
| `REDIS_URL` | `redis_url` | `""` | dispatch lock, translation cache, one lookup in `routers/workers.py`; all fail open if unset/unreachable |
| `RAZORPAY_TEST_KEY_ID` | `razorpay_key_id` | `""` | note the alias — `.env` key is `RAZORPAY_TEST_KEY_ID`, not `RAZORPAY_KEY_ID` |
| `RAZORPAY_TEST_KEY_SECRET` | `razorpay_key_secret` | `""` | same alias note |
| `RAZORPAY_WEBHOOK_SECRET` | `razorpay_webhook_secret` | `""` | |
| `RESEND_API_KEY` | `resend_api_key` | `""` | used by `services/notifications.py` to send email; if unset, emails are logged instead of sent |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_USERNAME` / `SMTP_PASSWORD` | `smtp_host` etc. | `smtp.gmail.com` / `587` / `""` / `""` | legacy SMTP fields, still read but Resend is the primary email path |
| `SMTP_FROM_EMAIL` | `smtp_from_email` | `noreply@kaargar.in` | |
| `MAP_BOX_API_KEY` | `mapbox_access_token` | `""` | **the active backend geocoding key.** `routers/geocode.py` wraps Mapbox's Geocoding v6 API (forward/reverse/autocomplete/place) — migrated off Google Places/Geocoding after it stopped working (billing/API-enablement drift on the old Google Cloud project). Raises 503 if unset. Same vendor as the frontend's `VITE_MAPBOX_TOKEN` (map tile rendering), now consolidated onto one account. **Earlier revisions of this table had this backwards — verified against the actual `geocode.py` code, not assumed.** |
| `GOOGLE_MAPS_API_KEY` | `google_maps_api_key` | `""` | **vestigial.** Defined in `config.py`'s `Settings` but not read anywhere else in the backend (verified by repo-wide grep) — a leftover from before the Mapbox migration above. Safe to remove. |
| `SARVAM_API_KEY` | `sarvam_api_key` | `""` | used by `services/translation.py` for chat-message translation (Sarvam AI); skipped if unset. `GROQ_API_KEY`/`groq_api_key` removed 2026-08-03 — translation.py fully migrated off Groq, the field and requirements.txt line were unused leftovers. |
| `PHONE_CALL_CIPHER_KEY` | `phone_call_cipher_key` | `""` | AES-256-GCM key, shared with frontend's `VITE_PHONE_CALL_KEY`, used to encrypt the masked phone number returned by `GET /jobs/{id}/contact` |

### Frontend (`FRONTEND/.env`)

| Var | Used in | Notes |
|---|---|---|
| `VITE_SUPABASE_URL` | `lib/supabase.js` | |
| `VITE_SUPABASE_ANON_KEY` | `lib/supabase.js` | |
| `VITE_API_URL` | `lib/api.js`, `components/kaargar/PuneMap.jsx` | backend base URL, e.g. `.../v1` |
| `VITE_RAZORPAY_KEY_ID` | `hooks/useRazorpay.js` | public/test key only |
| `VITE_MAPBOX_TOKEN` | `components/kaargar/PuneMap.jsx`, `JobTrackingMap.jsx`, `pages/job/SearchingPage.jsx` | **was missing from `.env.example` — now added.** Required for the tracking/search maps to render. |
| `VITE_PHONE_CALL_KEY` | `lib/phoneCipher.js` | **was missing from `.env.example` — now added.** Must match backend's `PHONE_CALL_CIPHER_KEY`. |

Both `.env.example` files have been corrected to include every variable above.

---

For everything else — the matching engine, booking flows, job lifecycle, penalty system, admin
config, commission/GST model, real-time channels, and known gaps — see **`SYSTEM_OVERVIEW.md`**.

---
> Source: [officialayush23/Kaargar](https://github.com/officialayush23/Kaargar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
