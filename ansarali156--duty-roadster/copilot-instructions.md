## duty-roadster

> > Operating manual for AI coding agents working in this repo. Read this **first**, before touching any code.

# AGENTS.md — Smart Bandobusth / Duty-Guard

> Operating manual for AI coding agents working in this repo. Read this **first**, before touching any code.

---

## 1. What this product is

**Smart Bandobusth** is a real-time GPS tracking and command-and-control system for police bandobast (deployment) operations.

- **End users:** ~7,000 uniformed police officers in the field carrying Android phones.
- **Control room:** Web dashboard for Admin / SP / DSP / CI / SI roles to monitor live officer positions, geofence breaches, and SOS events.
- **Stakes:** Officer life-safety, evidence chain-of-custody for court, public trust. A bug here can get a constable killed or get evidence thrown out by a magistrate. Treat every defect as a safety hazard, not a UX nit.

---

## 2. Repo layout

```
duty-guard/
├── AGENTS.md                   ← you are here
├── PLAN.md                     ← master remediation plan (read second)
└── smart-bandobusth/
    ├── README.md
    ├── backend/                ← FastAPI + PostgreSQL + Redis + Firebase FCM
    │   └── app/
    │       ├── main.py             # uvicorn entrypoint, lifespan, middleware
    │       ├── core/               # config, database, security (JWT, bcrypt)
    │       ├── routers/            # auth, officer, admin, supervisor, geofence, ws
    │       ├── services/           # gps_broadcaster, ws_manager, alert_service, cache_service, geofence
    │       ├── models/             # SQLAlchemy ORM
    │       ├── schemas/            # Pydantic
    │       └── migrations/         # Alembic
    └── frontend/               ← Capacitor (Android) WebView wrapping React/Vite + Leaflet
        └── src/
            ├── pages/              # Login, Dashboard, Live Map, Officer detail, etc.
            ├── store/              # Zustand stores (auth, location, alerts)
            ├── hooks/              # useOfflineSync, useGeolocation, useWebSocket
            └── components/
```

---

## 3. Architecture (as it stands today — see PLAN.md for the target)

```
┌─────────────────────┐         HTTPS              ┌────────────────────────────┐
│ Android (Capacitor) │ ──── POST /location-update│  FastAPI (single uvicorn   │
│  - WebView + React  │ ◄── WS  /ws/officer ──────│   process, asyncio,        │
│  - JS setInterval   │ ◄── WS  /ws/supervisor ───│   1 worker)                │
│  - IndexedDB queue  │                            │  - in-process WS dicts    │
└─────────────────────┘                            │  - GPSBroadcaster (20s)   │
                                                   │  - alert cascade inline   │
                                                   └──────┬──────────────┬─────┘
                                                          │              │
                                                   ┌──────▼─────┐  ┌─────▼─────┐
                                                   │ PostgreSQL │  │  Redis    │
                                                   │ + PostGIS  │  │ (Upstash) │
                                                   │ pool=20+30 │  │  cache +  │
                                                   └────────────┘  │  counter  │
                                                                   └───────────┘
                                                          │
                                                   ┌──────▼──────┐
                                                   │ Firebase    │
                                                   │ FCM         │
                                                   └─────────────┘
```

**Known load-bearing assumptions (undocumented before this audit):**
- Exactly **one** uvicorn worker. Bumping to 2+ silently corrupts WS fan-out and alert cascade.
- Single PG primary, no read replicas, no PgBouncer.
- Redis is best-effort — a Redis outage degrades silently (and in one case spams FCM, see PLAN.md §P0-4).
- All in-process state lives in `services/ws_manager.py` (`_connections`, `_role_index`, `_location_cache`).

---

## 4. Tech stack & versions

| Layer | Tech |
|---|---|
| Backend runtime | Python 3.11+, FastAPI, uvicorn (uvloop + httptools) |
| ORM / DB | SQLAlchemy 2.x async, asyncpg, Alembic, PostgreSQL 14+ with PostGIS |
| Cache / counters | Redis (Upstash optional) |
| Auth | JWT (HS256), bcrypt for PINs |
| Push | Firebase Admin SDK (FCM) |
| Mobile | Capacitor 5+, React 18, Vite, Zustand, Leaflet |
| Tests | pytest, pytest-asyncio (sparse coverage today) |
| Hosting (current) | Railway / Render (single container) |

---

## 5. Critical files — what NOT to touch without thinking twice

| File | Why it's load-bearing |
|---|---|
| `backend/app/services/ws_manager.py` | All in-process WS state. Changing the dict shapes silently breaks every consumer. |
| `backend/app/routers/officer.py` (POST /location-update) | The hot path — 600+ RPS at full load. Don't add synchronous I/O here. |
| `backend/app/services/gps_broadcaster.py` | Runs in the same event loop as ingest. Adding work here starves the API. |
| `backend/app/routers/auth.py` | Contains the **PIN backdoor (line 166)** and the dual-device session-token logic. |
| `backend/app/core/security.py` | JWT issue/verify. Falls back to a hardcoded dev secret if env unset (P0). |
| `backend/app/services/alert_service.py` | Alert cascade — inline FCM in the request path. Slow FCM = slow API. |
| `backend/app/main.py` `lifespan` | Runs DDL migrations as a background task at startup. Fragile. |

---

## 6. Coding conventions in this repo

- **Async everywhere** in backend; never call sync I/O from a route. CPU-bound work → `loop.run_in_executor`.
- **Raw SQL via `text()`** is used in several places (especially `auth.py`) to dodge schema-drift crashes — keep them parameterized; never f-string user input.
- **Pydantic v2** for schemas. Validators belong in `schemas/`, not in routes.
- **structlog** for logging — use `log.info("event_name", key=value)`, not f-strings.
- **No print()** in backend code.
- **Frontend:** functional components, hooks, Tailwind. Co-locate styles with components.
- **WCAG 2.2 AA** for the supervisor dashboard (Walmart standard, but also right for a govt product).

---

## 7. How to run locally

```bash
# Backend
cd smart-bandobusth/backend
uv venv .venv && source .venv/bin/activate
uv pip install --index-url https://pypi.ci.artifacts.walmart.com/artifactory/api/pypi/external-pypi/simple \
               --allow-insecure-host pypi.ci.artifacts.walmart.com -r requirements.txt
# Set env: DATABASE_URL, REDIS_URL, JWT_SECRET (REQUIRED), FCM_CREDENTIALS_JSON
alembic upgrade head
uvicorn app.main:app --reload --port 8000

# Frontend
cd smart-bandobusth/frontend
npm install
npm run dev          # web preview
npx cap sync android # build for device
```

---

## 8. Sub-agents on this project

| Agent | Owns |
|---|---|
| `code-reviewer` | Holistic code review — bugs, vulns, perf, design debt |
| `c-reviewer` | Hardcore systems / architecture review (concurrency, determinism, capacity) |
| `qa-kitten` | Test strategy, Playwright/Appium E2E, load profiles, chaos |
| `planning-agent` | Breaks down tasks into actionable steps |
| `tpm` | PRDs, stories, sprint tracking |
| `master-orchestrator-superagent` | Cross-domain orchestration for mobile + backend + tests |

**Orchestration rule:** Spawn at least two reviewers for any plan that touches the auth path, the ingest hot path, or the WS fan-out. Never trust a single critic for life-safety code.

---

## 9. Memory palace

This repo is indexed in `mempalace`. Use:
```bash
mempalace search "geofence flap"
mempalace search "PIN backdoor"
mempalace status
```
to recall context. Re-run `mempalace mine .` after large refactors so drawers stay current.

---

## 10. Read next

1. **PLAN.md** — the master remediation plan (P0 hot-fixes → architectural overhaul → rollout gates)
2. `smart-bandobusth/README.md` — the original product README
3. `smart-bandobusth/backend/app/routers/officer.py` — read the hot path before changing anything
4. `smart-bandobusth/backend/app/services/ws_manager.py` — understand the in-process state before scaling

---

*Maintained by jarvis (`code-puppy-c0fb22`). Bump version on architectural changes.*

---
> Source: [Ansarali156/duty_roadster](https://github.com/Ansarali156/duty_roadster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
