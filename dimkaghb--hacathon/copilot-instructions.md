## hacathon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Axel is a node-based AI video generation platform powered by Google Veo 3.1. Users create, extend, and chain videos through a visual React Flow canvas. The app is a full-stack monorepo: **Next.js 16 frontend** + **FastAPI backend** with Celery workers for async video generation.

## Commands

### Backend (run from `backend/`)

```bash
# Setup
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
alembic upgrade head

# Run API server
uvicorn app.main:app --reload --port 8000

# Run Celery worker (separate terminal)
celery -A app.core.celery_app worker --loglevel=info

# Alternative async workers
python -m app.workers.runner --type all

# Start infrastructure (PostgreSQL, Redis, Qdrant)
docker-compose up -d postgres redis qdrant

# Tests
pytest -v
pytest --cov=app tests/

# Linting
black .
isort .
mypy .
```

### Frontend (run from `frontend/`)

```bash
pnpm install
pnpm dev          # Dev server on localhost:3000
pnpm build        # Production build
pnpm lint         # ESLint
```

### Shell test scripts (from repo root)

```bash
./quick_test.sh                # Quick API validation (~2 min, no video gen)
./test_video_flow.sh           # Full E2E with actual video generation (~6-12 min)
./monitor_job.sh <job_id>      # Real-time job progress monitoring
```

## Architecture

```
Frontend (Next.js 16 + React Flow)
        │
   REST API + WebSocket
        │
Backend (FastAPI) ── Services Layer ── Celery Workers
        │                                    │
   PostgreSQL          Redis (broker)    Google Veo 3.1 API
                       Qdrant (vectors)  Google Cloud Storage
```

### Backend layers (`backend/app/`)

- **`api/`** — FastAPI route handlers. Routes: `/api/auth`, `/api/projects`, `/api/ai`, `/api/files`, `/api/characters`, `/api/subscriptions`, `/api/webhooks`. `ai.py` orchestrates video generation/extension/face analysis. `characters.py` provides user-level character library CRUD + wardrobe presets.
- **`services/`** — Business logic. `veo_service.py` wraps the Google Veo 3.1 API (generate, extend, poll). `face_service.py` handles face embedding extraction. `prompt_service.py` enhances prompts via Gemini. `storage_service.py` handles GCS uploads/downloads.
- **`workers/`** — Celery worker implementations (`video_worker.py`, `extension_worker.py`, `face_worker.py`). Inherit from `base.py` BaseWorker. Launched via `runner.py`.
- **`tasks/`** — Celery task definitions that dispatch to workers. `video_tasks.py` defines `generate_video` and `extend_video`.
- **`models/`** — SQLAlchemy ORM models with UUID primary keys. Key models: User, Project, Node, Connection, Job, Character, WardrobePreset. All use async sessions.
- **`schemas/`** — Pydantic request/response schemas mirroring models.
- **`core/`** — Infrastructure: `database.py` (async PostgreSQL via asyncpg), `celery_app.py`, `redis.py`, `security.py` (JWT + bcrypt).
- **`config.py`** — pydantic-settings `Settings` class. All config comes from environment variables / `.env` file.

### Frontend layers (`frontend/`)

- **`app/`** — Next.js pages: landing (`page.tsx`), login (`login/page.tsx`), main editor (`main/page.tsx`).
- **`components/canvas/`** — React Flow canvas. `ReactFlowCanvas.tsx` is the main component. Custom nodes in `nodes/` (VideoNode, PromptNode, ImageNode, ExtensionNode) and custom edges in `edges/`.
- **`components/landing/`** — Landing page section components (Hero, Features, FAQ, etc.).
- **`lib/api.ts`** — API client for all backend communication (HTTP + WebSocket).
- **`lib/contexts/AuthContext.tsx`** — Authentication state management, token storage in localStorage.

### Subscription & Credits (`backend/app/`)

- **`models/subscription.py`** — Subscription model (plan, status, credits_balance, trial dates, Polar IDs)
- **`models/credit_transaction.py`** — Append-only credit audit log
- **`models/polar_event.py`** — Webhook idempotency table
- **`services/subscription_service.py`** — Core credit logic: deduct (SELECT FOR UPDATE), refund, trial/activation/renewal
- **`services/polar_service.py`** — Polar.sh SDK wrapper (checkout, webhook verification)
- **`api/subscriptions.py`** — `/api/subscriptions` endpoints (status, checkout, trial, transactions)
- **`api/webhooks.py`** — `/api/webhooks/polar` (no auth, handles subscription lifecycle + order events)
- **`api/deps.py`** — `require_active_subscription` dependency (402 if no valid subscription)
- **`core/exceptions.py`** — `InsufficientCreditsError`

### Frontend subscription (`frontend/`)

- **`lib/contexts/SubscriptionContext.tsx`** — Subscription state + `useSubscription()` hook
- **`components/SubscriptionGate.tsx`** — Paywall overlay for gated content
- **`components/ui/CreditDisplay.tsx`** — Credit balance widget (sidebar)
- **`components/ui/CreditCostBadge.tsx`** — Inline credit cost badge on buttons
- **`app/pricing/page.tsx`** — Public pricing page
- **`app/subscription/success/page.tsx`** — Post-checkout redirect

### Key data flows

**Video generation:** Frontend POST `/api/ai/generate-video` → Credits deducted → Job created (PENDING) → Celery task dispatched → Worker calls Veo 3.1 → Polls for completion (10s interval, 360s max) → Downloads video → Uploads to GCS → Updates Job/Node status → Frontend polls `/api/ai/jobs/{id}`. On permanent failure, credits are refunded.

**Subscription lifecycle:** Register → Auto-trial (50 credits, 3 days) → Subscribe via Polar checkout → Webhook activates (300 credits) → Monthly renewal resets credits

**Credit costs:** Video gen standard=25, fast=10, extension standard=25, extension fast=10, face analysis=5, prompt enhancement=free

**Job status lifecycle:** PENDING → PROCESSING → COMPLETED/FAILED

**Node types:** PROMPT, IMAGE, VIDEO, EXTENSION, CONTAINER, RATIO, SCENE, CHARACTER, PRODUCT, SETTING

## Environment Variables

### Backend (`backend/.env`)
`DATABASE_URL`, `REDIS_URL`, `GEMINI_API_KEY`, `GCS_BUCKET`, `JWT_SECRET`, `GOOGLE_CLOUD_PROJECT`, `VEO_MODEL`
`POLAR_ACCESS_TOKEN`, `POLAR_WEBHOOK_SECRET`, `POLAR_PRO_PRODUCT_ID`, `POLAR_SANDBOX=true`, `POLAR_SUCCESS_URL`

### Frontend (`frontend/.env.local`)
`NEXT_PUBLIC_API_URL=http://localhost:8000`

## Infrastructure Dependencies

PostgreSQL 14+, Redis 6+, Qdrant (vector DB for face embeddings), Google Cloud Storage, Google Veo 3.1 API access.

## API Documentation

When the backend is running: Swagger UI at `/docs`, ReDoc at `/redoc`.

## Phase 1 Implementation Progress (Character Library, Product/Setting Nodes)

### Completed Backend Changes
- **New node types**: CHARACTER, PRODUCT, SETTING added to `NodeType` enum (models + schemas + migration)
- **Character model evolved**: Now user-scoped (`user_id` FK) instead of project-scoped. New fields: `source_images` (JSON array), `prompt_dna`, `voice_profile`, `performance_style`, `metadata`
- **WardrobePreset model**: `backend/app/models/wardrobe_preset.py` — outfit variations per character
- **Character Library API**: `backend/app/api/characters.py` mounted at `/api/characters` — full CRUD + wardrobe + face analysis
- **Schemas**: `backend/app/schemas/character.py`, `backend/app/schemas/wardrobe_preset.py`
- **Prompt context injection**: `build_product_context()`, `build_setting_context()`, `format_performance()` in `prompt_service.py`
- **Video task enrichment**: `video_tasks.py` now accepts `wardrobe_preset_id`, `product_data`, `setting_data` and builds enriched prompts
- **VideoGenerateRequest** updated with `wardrobe_preset_id`, `product_data`, `setting_data` fields
- **Migrations**: `20260211_add_character_product_setting_node_types.py`, `20260211_evolve_character_to_library.py`

### Completed Frontend Changes
- **CharacterNodeRF, ProductNodeRF, SettingNodeRF**: `frontend/components/canvas/nodes/` — 3 new React Flow node components
- **CharacterLibraryPanel**: `frontend/components/canvas/CharacterLibraryPanel.tsx` — slide-out picker for character library + wardrobe selection
- **characterLibraryApi**: Added to `frontend/lib/api.ts` — full CRUD + wardrobe + face analysis API client
- **NodeType updated**: `frontend/lib/types/node.ts` now includes `character | product | setting`
- **VideoNodeRF**: 5 input handles (prompt, image, character, product, setting) + status indicators for all connected inputs
- **ReactFlowCanvas**: `getConnectedData` extracts character/product/setting from connections; `handleGenerateVideo` passes all context to API
- **Floating dock**: Character (IconUser), Product (IconPackage), Setting (IconMapPin) added to dock items
- **Node types registry**: All 3 new types registered in `nodes/index.ts`

### Docker Commands
```bash
# Always apply migrations via docker (not local alembic)
docker exec videogen-api alembic upgrade head

# Rebuild after code changes
cd backend && docker-compose build api && docker-compose up -d api
```

## Phase 2 Implementation Progress (Scene Gallery, Templates, Timeline)

### Step 2.1 — Scene Gallery (Completed)
- **SceneDefinition model**: `backend/app/models/scene_definition.py` — scene types (hooks, body, closers)
- **Scene Definitions API**: `backend/app/api/scene_definitions.py` mounted at `/api/scene-definitions`
- **SceneGalleryPanel**: `frontend/components/canvas/panels/SceneGalleryPanel.tsx` — slide-out scene picker
- **Seeds**: `backend/app/seeds/scene_definitions.py` — 14 system scene definitions
- **Migration**: `backend/alembic/versions/20260211_add_scene_definitions.py`

### Step 5.2 — Campaign Organization (Completed)
- **Campaign model**: `backend/app/models/campaign.py` — campaigns with many-to-many junction tables (`campaign_projects`, `campaign_characters`)
- **Campaign schemas**: `backend/app/schemas/campaign.py` — CampaignCreate, CampaignUpdate, CampaignResponse, CampaignDetailResponse
- **Campaigns API**: `backend/app/api/campaigns.py` mounted at `/api/campaigns` — full CRUD + project/character assignment/removal
- **Migration**: `backend/alembic/versions/20260223_add_campaigns.py`
- **Frontend API client**: `campaignsApi` in `frontend/lib/api.ts` — list, getById, create, update, delete, addProject, removeProject, addCharacter, removeCharacter
- **Dashboard**: `frontend/app/dashboard/page.tsx` — Campaign section above projects with expand-to-see-projects, status badges (draft/active/archived), project/character counts, inline edit/delete/archive actions, assign-project dialog

### Step 2.2 — Multi-Node Templates (Completed)
- **Template model**: `backend/app/models/template.py` — template blueprints with graph_definition JSON
- **Template schemas**: `backend/app/schemas/template.py` — TemplateCreate, TemplateResponse, TemplateInstantiateRequest/Response
- **Templates API**: `backend/app/api/templates.py` mounted at `/api/templates` — CRUD + instantiate endpoint
- **Instantiate endpoint**: `POST /api/templates/{id}/instantiate` — stamps nodes/connections onto a project with variable substitution
- **Seeds**: `backend/app/seeds/templates.py` — 7 system templates (Product Testimonial, GRWM, Before/After, Unboxing, PAS, Day in the Life, Tutorial)
- **Migration**: `backend/alembic/versions/20260211_add_templates.py`
- **TemplateBrowserPanel**: `frontend/components/canvas/panels/TemplateBrowserPanel.tsx` — slide-out template browser with category tabs, search, variable input
- **Frontend API client**: `templatesApi` in `frontend/lib/api.ts` — list, getById, instantiate
- **ReactFlowCanvas**: "Templates" dock item replaces "Components", opens TemplateBrowserPanel, `handleInstantiateTemplate` stamps graph onto canvas

### Step 5.3 — Community Templates / Remix (Completed)
- **Template model extended**: Added `is_published`, `published_at`, `remix_count`, `rating`, `rating_count` columns
- **Template schema extended**: `TemplateResponse` includes community fields + `TemplateRateRequest` (1-5 stars)
- **Migration**: `backend/alembic/versions/20260223_add_community_template_fields.py`
- **New API endpoints** in `backend/app/api/templates.py`:
  - `GET /api/templates/community` — List published templates (sort by popular/recent/rating)
  - `GET /api/templates/mine` — List current user's templates
  - `POST /api/templates/{id}/publish` / `unpublish` — Toggle community visibility
  - `POST /api/templates/{id}/remix` — Clone template as user's own + award 1 credit to creator
  - `POST /api/templates/{id}/rate` — Rate template 1-5 stars (running average)
- **Subscription service**: `reward_credits()` method in `subscription_service.py` — awards bonus credits via ADJUSTMENT transaction
- **Frontend API client**: `templatesApi` extended with `listCommunity`, `listMine`, `publish`, `unpublish`, `remix`, `rate`
- **TemplateBrowserPanel**: Rewritten with 3 tabs (System / My Templates / Community), star ratings, remix button, publish/unpublish toggle, community sort (popular/recent/rating)

---
> Source: [Dimkaghb/Hacathon](https://github.com/Dimkaghb/Hacathon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
