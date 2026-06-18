## dealpulse

> > Last updated: 2026-03-22

# CLAUDE.md — DealPulse

> Last updated: 2026-03-22

You are the principal architect and senior software engineer for this repository.

## Project Identity

**Product Name:** DealPulse (Euron Course Project P3)
**GitHub:** `aiagentwithdhruv/dealpulse`
**Subdomain:** dealpulse.aiwithdhruv.com
**Type:** AI-powered real-time sales coaching & lead intelligence SaaS
**MVP Goal:** Ship a functional platform with live call transcription, real-time objection detection, AI coaching suggestions, deal scoring, pipeline analytics, and post-call reports.
**Full Vision:** 16-module sales intelligence engine with CRM sync, conversation intelligence, team performance, and ML-based deal prediction. See `docs/PRD.md`.
**Target Customers:** B2B sales teams, call centers, CRM platforms, sales agencies.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14+, TypeScript, Tailwind CSS |
| Backend | Python 3.11+, FastAPI, Pydantic v2 |
| Database | Supabase (PostgreSQL + pgvector) |
| Cache/Queue | Redis |
| AI (STT) | Deepgram Nova-2 (real-time streaming WebSocket) |
| AI (LLM) | OpenAI API (GPT-4o for reasoning, Haiku for classification) |
| AI (ML) | XGBoost (deal scoring, lead scoring — post-MVP) |
| AI Orchestration | LangGraph (multi-agent, typed state, supervisor) — post-MVP |
| Auth | Supabase Auth + JWT, RBAC |
| Deployment | Docker, AWS (ECS Fargate, ALB, S3, SQS) |

**Cost Targets:** AI coaching suggestion < $0.03 | Objection classification $0.001/segment | Deal scoring $0 (heuristic) | Post-call summary $0.05-$0.10 | Total per call ~$0.50-$0.80

---

## MVP Scope — What to Build NOW

### INCLUDE in MVP (Phase 1)

1. **Auth & RBAC** — Login/signup, role-based access (admin, manager, rep), JWT via Supabase Auth
2. **Call Transcription** — Deepgram streaming, speaker diarization, live transcript display, storage
3. **Live Call Coaching** — WebSocket-based real-time AI suggestions during calls, objection responses
4. **Objection Detection** — Haiku classifier on transcript chunks, 8 base objection types, confidence scoring
5. **Deal Scoring (Heuristic)** — Weighted signal scoring, factor attribution, per-call score updates
6. **Post-Call Reports** — AI summary, key moments, action items, talk-to-listen ratio, sentiment
7. **Sales Rep Dashboard** — Live call view (transcript, objections, suggestions, score), deal pipeline, metrics
8. **Manager Dashboard** — Team monitoring, pipeline health, rep performance, call review queue
9. **Pipeline Analytics** — Visual pipeline (Kanban + table), revenue forecast, velocity metrics, stuck deal alerts
10. **Call Recording** — S3 storage, replay with synchronized transcript
11. **Playbook Management** — Create/manage playbooks, coaching templates, objection response scripts
12. **Admin Panel** — User management, AI config, organization settings
13. **Basic Lead Scoring** — Score based on engagement and intent signals from calls
14. **Notifications** — In-app + email alerts for deal risks and stalled deals

### EXCLUDE from MVP (Post-launch — see docs/PRD.md for full 16 modules)

- CRM integration (Salesforce, HubSpot, Zoho)
- Advanced conversation intelligence (topic extraction, buying signals)
- Team performance dashboards and leaderboards
- Coaching insights and AI-generated recommendations
- Knowledge base RAG (battle cards during calls)
- Slack integration for alerts
- Custom ML scoring models (XGBoost)
- Mobile app (native iOS/Android)
- Churn prediction, multi-language, video calls
- Webhook outbound / SDK / Developer APIs

---

## Key Docs (Read Before Coding)

- `docs/PRD.md` — Product requirements (16 modules), user flows, milestones, pricing
- `docs/ARCHITECTURE.md` — Layers, data flows, AI/ML pipeline, key decisions
- `docs/API_SPEC.md` — 13 API sections, endpoints, schemas, WebSocket events, error responses
- `docs/DB_SCHEMA.md` — 25 tables, indexes, RLS policies, enums, relationship diagram
- `docs/DEPLOYMENT.md` — Docker, AWS ECS, CI/CD, environment variables, monitoring

---

## Architecture Overview

```
Clients: Sales Rep Dashboard + Manager Dashboard + Admin Panel (web)
              |
         ALB / API Gateway (WebSocket + REST)
              |
    +---------+---------+
    |                   |
Next.js App        FastAPI Backend
(Rep + Manager     (REST + WebSocket)
 + Admin UI)            |
    |          Services Layer
    |          (Call, Transcription, Objection,
    |           Coaching, DealScoring, Analytics,
    |           LeadScoring, Playbook, CRM, Auth)
    |                   |
    |          Repositories / Integrations
    |                   |
    +----> Supabase (PostgreSQL + pgvector)
                        |
              Redis (cache, pub/sub, queues)
                        |
              Deepgram (STT) + OpenAI (LLM)
                        |
              AWS S3 (recordings) + SQS (async jobs)
```

**One app, three role-gated views:**
- **Rep routes** (`/dashboard/*`): Live call view, deal pipeline, personal metrics, post-call analytics
- **Manager routes** (`/manager/*`): Team monitoring, pipeline health, rep performance, call review
- **Admin routes** (`/admin/*`): Organization setup, CRM config, user management, AI settings
- Role-based access via middleware; shared component library

---

## LangGraph Agents (5) — Post-MVP

| Agent | Role | Model | Triggers |
|-------|------|-------|----------|
| Transcription Agent | Process Deepgram stream, chunk, diarize, store | Deepgram Nova-2 | Audio stream active |
| Objection Detection Agent | Classify transcript segments for objections | Haiku | Every 3-5 seconds of transcript |
| Coaching Agent | Generate contextual response suggestions | GPT-4o | Objection detected (confidence > 0.7) |
| Deal Scoring Agent | Compute deal probability from multi-signal features | Heuristic / XGBoost | After each objection event or every 60s |
| CRM Sync Agent | Bidirectional sync with Salesforce/HubSpot/Zoho | N/A (API calls) | Post-call trigger or scheduled batch |

**State:** `CallState(TypedDict)` — call_id, deal_id, rep_id, transcript_chunks, objections, suggestions, deal_score, deal_score_factors, talk_to_listen_ratio, call_duration, errors.

**Key Patterns:**
1. **Dual-LLM Cost Optimization** — Haiku for classification ($0.001/call), GPT-4o only for coaching reasoning ($0.01-$0.03/suggestion)
2. **Heuristic Before ML** — Ship heuristic scoring immediately; collect labeled data for XGBoost training
3. **Audit Every AI Decision** — Log model, tokens, cost, latency, decision, confidence in `ai_decision_logs`
4. **Cache Common Responses** — Redis cache for coaching suggestions keyed by objection_type + context_hash (TTL 1h)

---

## Project Structure

### Backend (`/backend`)

```
backend/
  app/
    main.py                 # FastAPI app entry
    core/
      config.py             # Settings via env vars (pydantic-settings)
      security.py           # Auth, JWT validation, RBAC
      middleware.py          # CORS, logging, tenant context
      exceptions.py         # Centralized error handling
    routes/
      auth.py               # Login, signup, token refresh, invite
      calls.py              # Call CRUD, start/end, transcript, analytics
      deals.py              # Deal CRUD, pipeline, forecast, scoring
      leads.py              # Lead CRUD, scoring, conversion
      contacts.py           # Contact CRUD, interactions
      playbooks.py          # Playbook CRUD, steps management
      recordings.py         # Recording list, playback, bookmarks, search
      coaching.py           # AI coaching suggest, analyze, detect
      analytics.py          # Dashboard, pipeline, reps, objections, forecast
      crm.py                # CRM connections, sync, field mapping (Phase 2)
      admin.py              # Org settings, users, teams, AI config, audit logs
      notifications.py      # Notification list, mark read, preferences
      health.py             # Health check endpoints
      ws.py                 # WebSocket: live call stream, dashboard updates
    services/
      call_service.py       # Call lifecycle, recording management
      transcription_service.py # Deepgram streaming, chunk processing
      objection_service.py  # Objection detection pipeline
      coaching_service.py   # Coaching suggestion generation
      deal_scoring_service.py # Heuristic/ML deal scoring
      lead_scoring_service.py # Lead score computation
      analytics_service.py  # Dashboard aggregation, pipeline metrics
      playbook_service.py   # Playbook management logic
      recording_service.py  # S3 upload, presigned URLs, search
      crm_service.py        # CRM sync orchestration (Phase 2)
      notification_service.py # Multi-channel notification dispatch
      post_call_service.py  # Async post-call analytics (GPT-4o summary)
    repositories/
      call_repo.py
      transcript_repo.py
      deal_repo.py
      lead_repo.py
      contact_repo.py
      objection_repo.py
      suggestion_repo.py
      playbook_repo.py
      recording_repo.py
      analytics_repo.py
      notification_repo.py
      audit_repo.py
    models/                 # SQLAlchemy / Supabase models
    schemas/                # Pydantic request/response schemas
    integrations/
      deepgram_client.py    # Deepgram WebSocket streaming
      openai_client.py      # OpenAI API wrapper (completions + embeddings)
      supabase_client.py    # Supabase client init
      redis_client.py       # Redis connection + pub/sub
      s3_client.py          # AWS S3 operations
      sqs_client.py         # AWS SQS producer/consumer
      crm/                  # CRM adapters (Phase 2)
        base.py             # CRMProvider interface
        salesforce.py
        hubspot.py
        zoho.py
    workers/
      post_call_worker.py   # Async post-call analytics
      crm_sync_worker.py    # CRM sync consumer
      notification_worker.py # Notification dispatch
      scoring_worker.py     # Batch deal/lead score recalculation
    tests/
  requirements.txt
  Dockerfile
  alembic.ini
  alembic/
```

### Frontend (`/frontend`)

```
frontend/
  app/
    layout.tsx              # Root layout
    page.tsx                # Landing / redirect
    (auth)/
      login/page.tsx
      signup/page.tsx
      invite/page.tsx
    (dashboard)/            # Sales Rep views
      page.tsx              # Rep home (active deals, recent calls)
      calls/page.tsx        # Call history
      calls/[id]/page.tsx   # Call detail + post-call analytics
      calls/[id]/live/page.tsx # Live call view (transcript, coaching, score)
      deals/page.tsx        # Deal pipeline (Kanban + table)
      deals/[id]/page.tsx   # Deal detail (score, timeline, calls)
      leads/page.tsx        # Lead list
      leads/[id]/page.tsx   # Lead detail
      recordings/page.tsx   # Recording library
      recordings/[id]/page.tsx # Recording playback + transcript
      playbooks/page.tsx    # Playbook list
    (manager)/              # Manager views
      page.tsx              # Manager home (team overview)
      team/page.tsx         # Team performance
      pipeline/page.tsx     # Pipeline analytics
      calls/page.tsx        # Call review queue
      coaching/page.tsx     # Coaching insights
    (admin)/
      page.tsx              # Admin home (org overview)
      users/page.tsx        # User management
      teams/page.tsx        # Team management
      integrations/page.tsx # CRM config (Phase 2)
      playbooks/page.tsx    # Playbook management
      settings/page.tsx     # AI config, org settings
      audit/page.tsx        # Audit logs
  components/
    ui/                     # Design system primitives (Button, Card, Input, Badge, etc.)
    layout/                 # Sidebar, Header, Footer, PageShell
    calls/                  # LiveCallView, TranscriptPanel, CoachingPanel, CallCard
    deals/                  # DealCard, DealPipeline, DealScoreIndicator, DealTimeline
    leads/                  # LeadCard, LeadScoreBar, LeadList
    analytics/              # PipelineChart, ForecastCard, RepScorecard, MetricCard
    recordings/             # RecordingPlayer, TranscriptSync, BookmarkList
    playbooks/              # PlaybookEditor, StepList, ObjectionResponseEditor
    admin/                  # UserTable, TeamManager, AIConfigPanel
    notifications/          # NotificationCenter, AlertBanner
  hooks/
    useWebSocket.ts         # Live call WebSocket hook
    useDashboardWS.ts       # Dashboard updates WebSocket hook
    useAuth.ts              # Auth state
    useDeals.ts             # Deal data fetching
    useCalls.ts             # Call data fetching
    useAnalytics.ts         # Analytics data fetching
  lib/
    api.ts                  # API client (fetch wrapper with auth)
    constants.ts            # App constants, routes, enums
    utils.ts                # Utility functions
    formatters.ts           # Number, date, duration formatters
  services/
    auth.ts
    calls.ts
    deals.ts
    leads.ts
    analytics.ts
    playbooks.ts
    recordings.ts
  types/
    index.ts                # Shared TypeScript types
  public/
  tailwind.config.ts
  next.config.ts
  tsconfig.json
  package.json
  Dockerfile
```

---

## Design System — "Euron" (Strict)

All UI must follow this system. No deviations.

### Color Palette

| Token | Value | Usage |
|-------|-------|-------|
| `primary` | `#0A66C2` | Buttons, links, active states, brand accent |
| `primary-hover` | `#004182` | Hover/active on primary elements |
| `bg` | `#F3F6F8` | Page background |
| `surface` | `#FFFFFF` | Cards, panels, modals |
| `border` | `#E5E7EB` | Card borders, dividers, input borders |
| `text-primary` | `#111827` | Headings, body text |
| `text-muted` | `#6B7280` | Secondary text, captions, placeholders |
| `success` | `#057642` | Success states, positive score trends |
| `warning` | `#B45309` | Warning states, at-risk deals |
| `error` | `#B91C1C` | Error states, declining scores |
| `input-border` | `#D1D5DB` | Default input borders |

Blue is the dominant color. Success/warning/error used sparingly and functionally.

### Typography

- **Font:** Inter (import from Google Fonts)
- **Headings:** weight 600-700
- **Body:** weight 400-500

| Element | Size | Weight |
|---------|------|--------|
| Page title | 28-32px | 700 |
| Section header | 20-24px | 600 |
| Card title | 16-18px | 600 |
| Body text | 14-16px | 400 |
| Caption / meta | 12px | 400 |

Line height: 1.4-1.6 for all text.

### Cards

```css
background: #FFFFFF;
border: 1px solid #E5E7EB;
border-radius: 8px;
box-shadow: none; /* flat, stable, professional */
```

### Buttons

**Primary:** `bg: #0A66C2, color: #FFF, border-radius: 999px (pill), font-weight: 600`
**Secondary:** `bg: transparent, border: 1px solid #0A66C2, color: #0A66C2, pill`
**Tertiary:** Text only, muted gray.

### Forms & Inputs

- Height: 40-44px | Border: 1px solid #D1D5DB | Focus: border #0A66C2 + `ring-1 ring-blue-500/20`
- Labels above inputs | Placeholder in muted gray

### Icons & Motion

- Outline/stroke-based only (Lucide React or Heroicons outline)
- Hover transitions: 100-150ms ease | No bounce, no flashy animations

### Tailwind Config

```js
colors: {
  brand: { DEFAULT: '#0A66C2', hover: '#004182' },
  surface: '#FFFFFF',
  bg: '#F3F6F8',
  border: '#E5E7EB',
  'input-border': '#D1D5DB',
  'text-primary': '#111827',
  'text-muted': '#6B7280',
  success: '#057642',
  warning: '#B45309',
  error: '#B91C1C',
}
```

---

## Database Schema (MVP subset)

| Table | Purpose |
|-------|---------|
| `tenants` | Multi-tenant org isolation |
| `users` | Admin, manager, rep identity |
| `teams` | Team hierarchy (manager -> reps) |
| `calls` | Sales call records with metadata |
| `call_transcripts` | Chunked real-time transcripts |
| `call_segments` | Speaking segments for talk-to-listen ratio |
| `deals` | Sales opportunities with pipeline stage |
| `deal_scores` | Historical deal score snapshots |
| `deal_history` | Deal stage change audit trail |
| `leads` | Prospective leads before conversion |
| `lead_scores` | Historical lead score snapshots |
| `contacts` | People associated with deals/leads |
| `contact_interactions` | All interactions with a contact |
| `playbooks` | Sales coaching playbooks |
| `playbook_steps` | Individual steps within playbooks |
| `coaching_suggestions` | AI-generated coaching suggestions |
| `objection_detections` | Detected objections from calls |
| `crm_connections` | CRM config per org (Phase 2) |
| `crm_sync_logs` | CRM sync history |
| `recordings` | Call recording metadata (S3 refs) |
| `analytics_snapshots` | Pre-computed analytics aggregates |
| `notifications` | In-app notifications |
| `audit_logs` | User action audit trail |
| `api_keys` | Developer API key management |
| `ai_decision_logs` | AI decision audit trail (compliance, cost) |

Full schema with 25 tables, enums, indexes, RLS: see `docs/DB_SCHEMA.md`.

**Conventions:** UUID PKs, timestamptz, organization_id + RLS, enums for stages/status/roles/types.

---

## API Endpoints (MVP subset)

**Base:** `/api/v1`

| Area | Key Endpoints |
|------|--------------|
| Auth | `POST /auth/login`, `/auth/signup`, `/auth/refresh`, `/auth/invite` |
| Calls | `GET/POST /calls`, `POST /calls/{id}/start`, `POST /calls/{id}/end`, `GET /calls/{id}/transcript`, `GET /calls/{id}/analytics` |
| WebSocket | `WS /ws/calls/{call_id}/live` (transcript, objections, suggestions, score), `WS /ws/dashboard` |
| Deals | `GET/POST /deals`, `PATCH /deals/{id}`, `GET /deals/{id}/score`, `GET /deals/pipeline`, `GET /deals/forecast` |
| Leads | `GET/POST /leads`, `GET /leads/{id}/score`, `POST /leads/{id}/convert` |
| Contacts | `GET/POST /contacts`, `GET /contacts/{id}/interactions` |
| Playbooks | `GET/POST /playbooks`, `POST /playbooks/{id}/steps` |
| Recordings | `GET /recordings`, `GET /recordings/{id}/playback`, `GET /recordings/search` |
| AI/Coaching | `POST /ai/coaching/suggest`, `POST /ai/scoring/deal`, `POST /ai/detect-objections` |
| Analytics | `GET /analytics/dashboard`, `/analytics/pipeline`, `/analytics/reps`, `/analytics/forecast` |
| CRM (v2) | `POST /crm/connections`, `POST /crm/sync`, `GET /crm/sync/logs` |
| Admin | `GET/PATCH /admin/organization`, `GET /admin/users`, `GET/PATCH /admin/config/ai` |
| Health | `GET /health`, `GET /health/detailed` |

Full API (13 sections, 90+ endpoints): see `docs/API_SPEC.md`.

---

## Key Flows (MVP)

### 1. Rep starts a live call with coaching
```
Rep selects deal -> Clicks "Start Call" -> POST /calls/{id}/start
-> WebSocket /ws/calls/{call_id}/live opens
-> Backend opens Deepgram stream -> Audio transcribed in real-time
-> Transcript chunks pushed to rep via WebSocket
-> Every 3-5s: objection detection (Haiku) on transcript window
-> If objection (confidence > 0.7): coaching suggestion generated (GPT-4o)
-> Suggestion pushed to rep via WebSocket (<2s latency)
-> Deal score updated and pushed via WebSocket
```

### 2. Post-call analytics
```
Rep ends call -> POST /calls/{id}/end
-> Deepgram stream closed -> Recording uploaded to S3
-> Post-call analytics job enqueued (SQS)
-> Worker: GPT-4o generates summary, key moments, action items
-> Report stored in DB -> Available in <30s
-> Deal score recalculated with final call data
```

### 3. Manager reviews pipeline
```
Manager opens Pipeline Analytics -> GET /analytics/dashboard
-> Visual pipeline: deals by stage, weighted revenue, risk flags
-> Clicks at-risk deal -> GET /deals/{id} + /deals/{id}/score
-> Reviews call history, objection patterns, score trend
-> Prepares coaching notes for 1:1
```

### 4. Deal scoring pipeline
```
Call event (objection, engagement signal) OR scheduled batch
-> Deal Scoring Service gathers feature vector
-> Heuristic: weighted signals (objections, engagement, recency, stage)
-> Score (0-100) + risk factors computed
-> Stored in deal_scores + deal.current_score updated
-> If below threshold: notification triggered
```

---

## Build Order

### Phase 1 — Foundation (Week 1-2)
1. Project scaffolding (backend + frontend)
2. Supabase setup + migrations (core tables: tenants, users, teams, deals, calls)
3. Auth (login/signup/JWT/RBAC)
4. Health check endpoint
5. Basic layout shell (sidebar, header, routing, role-gated)
6. Deepgram integration (basic streaming test)

### Phase 2 — Real-Time Core (Week 3-4)
7. WebSocket infrastructure (live call endpoint)
8. Deepgram streaming proxy (backend -> Deepgram -> frontend)
9. Live transcription display (speaker diarization)
10. Objection detection pipeline (Haiku classifier)
11. Coaching suggestion engine (GPT-4o)
12. Deal CRUD + basic pipeline view

### Phase 3 — Intelligence (Week 5-6)
13. Deal scoring (heuristic weighted signals)
14. Post-call analytics (async GPT-4o summarization)
15. Pipeline analytics dashboard
16. Call recording (S3 upload + playback)
17. Lead CRUD + basic scoring

### Phase 4 — Management & Polish (Week 7-10)
18. Playbook CRUD + step management
19. Notification system (in-app + email)
20. Manager dashboard (team monitoring, call review)
21. Recording library (search, bookmarks, share)
22. Admin panel (user management, AI config, org settings)
23. Error handling, loading states, empty states
24. Basic tests for services and auth
25. Docker setup + deployment config

---

## Coding Conventions

### Backend (Python/FastAPI)
- **Architecture:** Routes > Services > Repositories (clean layered)
- Routes are thin — validation + dependency injection only
- Business logic in services, DB access through repositories
- Pydantic v2 for all request/response schemas
- Async I/O for all external calls (Deepgram, OpenAI, Supabase, Redis, S3)
- Error response: `{"code": "...", "message": "...", "details": ...}`
- Structured JSON logging with `request_id`, `call_id`, `deal_id`, `user_id`
- Never hardcode secrets

### Frontend (Next.js/TypeScript)
- App Router (Next.js 14+), TypeScript strict mode
- Small, reusable components; separate presentation from logic
- Handle loading, error, and empty states on every page
- WebSocket for live call data via `useWebSocket` hook
- Role-based routing middleware (rep, manager, admin)
- API calls through centralized `lib/api.ts`

### Security
- No secrets in code, Docker layers, or client bundles
- Validate all user input server-side
- RBAC enforced at API layer (middleware + route decorators)
- RLS enabled on all tenant-scoped tables
- Treat audio streams, transcripts, and prompts as untrusted
- PII masking on transcript storage

### Testing
- Tests for all service methods and auth flows
- Mock external services (Deepgram, OpenAI, Supabase, S3) in tests
- Cover validation, failure, and edge cases
- Integration tests for WebSocket call flow

---

## Environment Variables

```env
# Supabase
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
DATABASE_URL=

# Deepgram
DEEPGRAM_API_KEY=

# OpenAI
OPENAI_API_KEY=

# Redis
REDIS_URL=

# AWS S3
AWS_S3_BUCKET=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=

# AWS SQS
SQS_POST_CALL_QUEUE_URL=
SQS_CRM_SYNC_QUEUE_URL=

# App
APP_ENV=development
API_BASE_URL=http://localhost:8000
FRONTEND_URL=http://localhost:3000
JWT_SECRET=

# Sentry
SENTRY_DSN=
```

---

## Development Workflow

1. **Start backend:** `cd backend && uvicorn app.main:app --reload --port 8000`
2. **Start frontend:** `cd frontend && npm run dev`
3. **Start Redis:** `docker run -p 6379:6379 redis:alpine`
4. **Run migrations:** `supabase db push`
5. **Run tests:** `cd backend && pytest` / `cd frontend && npm test`
6. **Docker (full stack):** `docker-compose up`

---

## Prompt Persistence Rule

Every prompt must be saved to `prompts/prompt-history.md`:
- Format: ISO 8601 timestamp + exact prompt text
- Append only, never delete existing entries
- Never save secrets/tokens

---

## References

- PRD (16 modules): `docs/PRD.md`
- Architecture: `docs/ARCHITECTURE.md`
- API Spec (13 sections): `docs/API_SPEC.md`
- DB Schema (25 tables): `docs/DB_SCHEMA.md`
- Deployment: `docs/DEPLOYMENT.md`
- Maps to: Onsite Sales Intelligence, QuotaHit lead scoring pipeline
- Engineering Rules: `cursor-rules/claude/CLAUDE.md`

---
> Source: [aiagentwithdhruv/dealpulse](https://github.com/aiagentwithdhruv/dealpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
