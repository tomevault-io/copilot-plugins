## magic-grimoire

> AI-powered Magic: The Gathering deck generator. Users describe a deck in natural language; the system generates a balanced 60-card deck using real MTG cards via Anthropic Claude + Scryfall.

# CLAUDE.md — Magic Grimoire

AI-powered Magic: The Gathering deck generator. Users describe a deck in natural language; the system generates a balanced 60-card deck using real MTG cards via Anthropic Claude + Scryfall.

---

**Documentation workflow:** see `docs/documentation-guide.md` for how to generate Obsidian notes, ADR conventions, and folder structure.

---

## Repo Layout

```
magic-grimoire/
├── apps/
│   ├── api-server/     # FastAPI + Celery (Python 3.13)
│   └── web-app/        # Next.js 15 + TypeScript
├── packages/           # Shared Node packages (empty — placeholder for future)
├── docs/               # Obsidian knowledge base (see docs/CLAUDE.md)
├── .claude/            # Claude Code skills (/adr, /prd)
├── .obsidian/          # Obsidian vault config
├── .github/            # CI workflows
├── Makefile            # Unified entry point: make dev / test / build / lint
└── docker-compose.yml
```

---

## Backend (`apps/api-server/`)

**Entry point:** `apps/api-server/app/main.py` → mounts all routers via `router.py`

**Structure:**
```
apps/api-server/app/
├── core/
│   ├── config.py       # Settings from env vars (Pydantic BaseSettings)
│   └── database.py     # Async SQLAlchemy engine + session factory
├── services/
│   ├── llm/            # LLM abstraction: claude.py, ollama.py, factory.py
│   ├── scryfall_service.py  # Scryfall API: search_cards(), enrich_cards()
│   └── redis_cache.py       # Redis get/set helpers
├── decks/
│   ├── model.py    # Deck ORM model
│   ├── routes.py   # POST /decks/generate, GET /decks, GET /decks/:id, DELETE /decks/:id
│   ├── dtos.py     # Request/response Pydantic schemas
│   └── worker.py   # Celery task: generate_deck_task()
├── tasks/
│   ├── model.py    # Task ORM model
│   ├── routes.py   # GET /tasks/:id/stream (SSE)
│   └── dtos.py
└── workers/
    └── celery_app.py   # Celery app instance + broker config
```

**Key pattern — deck generation pipeline (Celery worker):**
1. `parse_intent(prompt)` → LLM → colors, creature_types, keywords, strategy
2. `search_cards(intent)` → Scryfall API (Redis-cached, 24h TTL)
3. `compose_deck(intent, cards, format)` → LLM → 60-card list
4. `enrich_cards(cards)` → Scryfall API per card (Redis-cached)
5. Save Deck + Task to PostgreSQL, publish `completed` to Redis Pub/Sub

Each step publishes a progress event to Redis Pub/Sub (`task:{id}`) which is streamed to the client via SSE at `GET /tasks/:id/stream`.

**Database:** PostgreSQL 16, async via `asyncpg`. Models in `*/model.py`. Migrations in `alembic/`.

**Auth:** JWT from Supabase, validated in middleware. `user_id` extracted from token.

**Package manager:** `uv`. Dependencies in `pyproject.toml`.

---

## Frontend (`apps/web-app/`)

**Framework:** Next.js 15 App Router, TypeScript, Tailwind CSS 4.

```
apps/web-app/app/
├── layout.tsx              # Root layout with SpineNav + UserProvider
├── page.tsx                # Landing page (Hero, Ritual, Features, Marquee, CTA)
├── grimoire/               # Deck generation page (split-screen chat + deck panel)
├── library/                # Saved decks (auth-gated)
├── context/
│   └── UserContext.tsx     # localStorage-backed user state
├── components/
│   ├── SpineNav.tsx         # Fixed left sidebar navigation
│   ├── ArcaneSigil.tsx      # Animated rotating rings SVG
│   ├── ManaSymbol.tsx       # MTG mana cost display
│   ├── DeckPanel.tsx        # Right panel: deck list/grid view
│   ├── AuthModal.tsx        # Login/signup modal
│   └── ArcaneSigilLogo.tsx            # SealLogo, Ornament, Frame
└── hooks/
    ├── useReveal.ts         # Intersection observer reveal animation
    └── useAutoScroll.ts     # Auto-scroll on new content
```

**Auth:** Supabase Auth (Google + GitHub OAuth). JWT passed as `Authorization: Bearer` header to the backend.

**Real-time:** Connects to `GET /tasks/:id/stream` (SSE) to receive deck generation progress events. No polling.

---

## Infrastructure

| Service | Port | Purpose |
|---|---|---|
| FastAPI | 8000 | REST API |
| Next.js | 3000 | Frontend |
| PostgreSQL | 5432 | Primary DB |
| Redis | 6379 | Celery broker + cache + Pub/Sub |
| Flower | 5555 | Celery monitoring UI |

**Local dev:** `docker-compose.yml` at repo root. All services have health checks. Run with `make dev`.

**CI:** `.github/workflows/` — `api-server.yml` (pytest, paths: `apps/api-server/**`), `web-app.yml` (lint + build, paths: `apps/web-app/**`), `gitleaks.yml` (secret scan).

---

## Key Conventions

- **Python:** async throughout. Never use sync SQLAlchemy calls inside async routes.
- **DTOs:** Pydantic schemas in `dtos.py`, never expose ORM models directly.
- **Redis cache keys:** `scryfall:search:{query}` and `scryfall:card:{name}`. TTL = 24h.
- **Task status values:** `queued` → `processing` → `completed` | `failed`
- **Deck status values:** `pending` → `processing` → `completed` | `failed`
- **Env vars:** defined in `apps/api-server/app/core/config.py`. Backend `.env` at `apps/api-server/.env`.

---
> Source: [RobertGleison/magic-grimoire](https://github.com/RobertGleison/magic-grimoire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
