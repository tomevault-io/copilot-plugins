## ai-study-architect

> **Study Architect** by Quantelect — Mastery-based AI study companion that proves you learned it. CS education beachhead. Claude API (primary) + OpenAI (fallback).

# CLAUDE.md

## Project Overview

**Study Architect** by Quantelect — Mastery-based AI study companion that proves you learned it. CS education beachhead. Claude API (primary) + OpenAI (fallback).

**Live**: https://aistudyarchitect.com
**Design**: [design/DESIGN.md](design/DESIGN.md) (canonical tokens, typography, glow recipes)
**Strategy**: [docs/direction/NEW_DIRECTION_2025.md](docs/direction/NEW_DIRECTION_2025.md)
**Build plan**: [docs/plans/2026-03-14-001-feat-full-product-build-phases-neg1-0-1-plan.md](docs/plans/2026-03-14-001-feat-full-product-build-phases-neg1-0-1-plan.md)
**Brainstorm**: [docs/brainstorms/2026-03-13-mvp-frontend-brainstorm.md](docs/brainstorms/2026-03-13-mvp-frontend-brainstorm.md)

## Development Rules

### NO EMOJIS in Code or Terminal Output
- Windows terminals fail with Unicode emojis (UnicodeEncodeError)
- Use ASCII: [PASS], [FAIL], [WARN], SUCCESS:, ERROR:
- Exception: robot emoji in Claude attribution commit trailers

### Frontend Styling Rules
- **New components**: Tailwind v4 + shadcn/ui (Radix) + Lucide icons
- **Legacy components** (chat, content): MUI + Emotion — Phase 3 removal pending
- Design tokens from `@theme` in `src/index.css` — never hardcode hex colors
- `cn()` from `@/lib/utils` for conditional class merging
- Tailwind class order: sorted by prettier-plugin-tailwindcss
- `npx shadcn@latest add <component>` — add one at a time (batch silently fails)

## Common Commands

### Backend
```bash
cd backend
uvicorn app.main:app --reload                  # Dev server (port 8000)
pytest tests/ -v                                # Run tests
ruff check app/ --fix                          # Lint + fix
alembic upgrade head                           # Run migrations
alembic revision --autogenerate -m "message"  # New migration (needs local PG running)
```

### Frontend
```bash
cd frontend
npm run dev                                    # Dev server (port 5173)
npm test                                       # Vitest
npm run typecheck                             # TypeScript check
npm run lint                                  # ESLint
npm run build                                 # Production build (tsc && vite build)
# Tailwind v4: @tailwindcss/vite plugin, no tailwind.config needed
# shadcn/ui: npx shadcn@latest add <component>
```

### Quality Checks
```bash
# Backend full check
cd backend && ruff check app/ && mypy app/ && pytest tests/ -v

# Frontend full check
cd frontend && npm run lint && npm run typecheck && npm test

# E2E
cd frontend && npx playwright test
```

### Deploy
```bash
cd worker && npx wrangler deploy               # Backend (CF Container)
# Frontend: auto-deploys from GitHub via Vercel
```

## Architecture

### Stack
- **Frontend**: React 18 + TypeScript + Vite 6 + Tailwind v4 + shadcn/ui → Vercel
- **Backend**: FastAPI → Cloudflare Container (Docker, 1/4 vCPU, 1 GiB)
- **Worker**: CF Worker routes `/api/*` → Container, everything else → Vercel proxy
- **Database**: Neon PostgreSQL (serverless), Alembic migrations
- **Cache**: Upstash Redis (REST API, _NoOpCache fallback)
- **Storage**: Cloudflare R2 (file uploads + backups)
- **AI**: Claude API (streaming SSE) → OpenAI fallback. Direct SDKs, no LangChain.

### Backend Structure
```
backend/app/
├── api/v1/
│   ├── api.py              # Main router (12 sub-routers)
│   ├── auth.py, chat.py, tutor.py, content.py, concepts.py
│   ├── subjects.py         # Subject CRUD
│   ├── study_sessions.py   # Session lifecycle (start/pause/resume/stop)
│   ├── dashboard.py        # Dashboard summary (3-query pattern)
│   ├── admin.py, agents.py, csrf.py, websocket.py
│   └── endpoints/backup.py
├── agents/                  # Lead tutor (Socratic questioning)
├── core/                    # config, security, database, csrf, cache, rate_limiter, rsa_keys, utils
├── models/                  # user, content, study_session, subject, practice, chat_message, concept, user_concept_mastery
├── schemas/                 # Pydantic v2 schemas (UUID-based, model_config)
└── services/                # ai_service_manager, claude_service, openai_fallback, content_processor, concept_extraction, storage
```

### Frontend Structure
```
frontend/src/
├── app/layout/              # AppShell, TopNav (Tailwind)
├── pages/                   # DashboardPage, StudyPage, FocusPage, ContentPage
├── components/
│   ├── dashboard/           # HeroMetrics, SubjectList, ContributionHeatmap, StartFocusCTA (Tailwind + SVG)
│   ├── ui/                  # shadcn/ui primitives (button, card, dialog, dropdown-menu, input, tabs, badge, tooltip, sonner)
│   ├── auth/                # LoginForm, RegisterForm, ProtectedRoute (Tailwind + shadcn)
│   ├── chat/                # ChatInterface (MUI — legacy, Phase 3 restyle)
│   └── content/             # ContentUpload, ContentList, ContentSelector (MUI — legacy)
├── hooks/useTimer.ts        # Web Worker timer hook
├── workers/timer.worker.ts  # Date.now()-based background timer
├── contexts/AuthContext.tsx  # Global auth state
├── services/                # api.ts (Axios), auth.service.ts, tokenStorage.ts (legacy cleanup stub)
├── lib/utils.ts             # cn() helper (clsx + tailwind-merge)
└── index.css                # @fontsource imports (layer(base)) → @import "tailwindcss" → @theme tokens
```

### Routing
- `/login`, `/register` — Auth (GuestRoute, no shell)
- `/` — Dashboard (ProtectedRoute, AppShell)
- `/study` — Chat/tutor (ProtectedRoute, AppShell)
- `/focus` — Focus timer (ProtectedRoute, AppShell)
- `/content` — Content management (ProtectedRoute, AppShell)
- `/subjects/:id` — Subject Detail (ProtectedRoute, AppShell)

### Key Patterns
- **Session state machine**: in_progress/paused/completed/cancelled, atomic transitions, partial unique index (one active per user)
- **Dashboard 3-query**: 28-day aggregation + active session check + streak calculation. Python timezone computation (not `func.timezone()` — crashes on Neon).
- **AI fallback**: Claude available? → Claude stream. No? → OpenAI response. Runtime key validation.
- **Rate limiter**: Single shared instance in `app/core/rate_limiter.py` (import as `limiter`, not `shared_limiter`)
- **CSS import order**: @fontsource `layer(base)` FIRST → `@import "tailwindcss"` SECOND → `@theme` tokens. Wrong order = invisible fonts.
- **Agent state**: Redis (2h TTL) + 50-agent local LRU cache
- **Content search**: PostgreSQL full-text search (tsvector + GIN index, `to_tsquery` with `:*` prefix matching + `ts_rank`). Weighted: title(A) > description(B) > extracted_text(C). Non-alnum chars stripped via `re.sub`. Trigger auto-updates on INSERT/UPDATE.
- **Security**: JWT RS256 (HS256 fallback), CSRF double-submit, SecurityHeaders middleware, httpOnly cookies (no tokens in response body), refresh token rotation with Redis family tracking

## CI/CD

- **staging.yml**: Tests on push to develop/staging, PR to main. PostgreSQL 17 service container. Node 20, checkout@v6, setup-python@v6, setup-node@v6.
- **deploy.yml**: Security scan (semgrep via pipx — NOT pip), migrations, CF Worker deploy. Node 20, same action versions.
- **claude-code-review.yml**: AI PR review with Claude Sonnet.
- **claude.yml**: Claude Code agent (@claude mentions in issues/PRs).
- **backup.yml**: Automated database backups.

## Critical Reminders

### Must-Know
- **Worker MUST NOT strip /api prefix** — backend expects full paths
- **API docs blocked** — Worker returns 404 for /api/docs, /api/openapi.json, /api/redoc
- **Neon migrations** — Use direct (non-pooled) connection for `alembic upgrade head`
- **API keys at runtime** — Services must check keys at runtime, not import time
- **NEVER change BACKUP_ENCRYPTION_KEY** — Loses access to all previous backups
- **CSP worker-src** — Production must be `'self' blob:` (not `'none'`) for Web Workers
- **MUI still installed** — Bundled as `mui-vendor` chunk in vite.config.ts rollupOptions. Removal in Phase 3.
- **RSA keys from env vars** — Loaded from `RSA_PRIVATE_KEY`/`RSA_PUBLIC_KEY` CF Worker secrets (base64-encoded PEM). Generate with `scripts/export_rsa_keys_b64.py`, store via `wrangler secret put`. `rotate_keys()` is in-memory only when env vars active.
- **Use `utcnow()` not `datetime.now(UTC)`** — All `DateTime` columns are timezone-naive. `datetime.now(UTC)` returns tz-aware, causing `TypeError` on arithmetic with DB-loaded values. Import from `app.core.utils`.

### Windows Dev
- PostgreSQL on port 5433 (not 5432). Start: `pg_ctl -D "C:/Program Files/PostgreSQL/17/data" start`
- Use `venv\Scripts\python.exe` for virtual environment
- `python-magic-bin` may need uncommenting in requirements.txt

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Empty AI responses | Use @property for runtime API key loading + streaming wrapper |
| CSRF 403 on API calls | JWT endpoints exempted in `app/core/csrf.py` |
| Redis connection fails | _NoOpCache automatically takes over |
| Frontend 404 on direct route | CF Worker proxies non-API to Vercel (SPA routing) |
| Tailwind classes not applying | Ensure `@import "tailwindcss"` in index.css, `@tailwindcss/vite` in vite.config.ts |
| shadcn component not found | Add one at a time: `npx shadcn@latest add <component>`. Check components.json. |
| Focus timer blocked in prod | Update `worker-src` in security_headers.py to `'self' blob:` |
| Fonts not rendering | @fontsource imports must come BEFORE Tailwind import with `layer(base)` |
| Dashboard 500 on Neon | Don't use `func.timezone()` — compute timezone boundaries in Python |
| Import `shared_limiter` fails | The export is `limiter` from `app.core.rate_limiter` |
| Streaming not working | Check SSE in ai_service_manager. OpenAI streaming NOT implemented yet. |
| Vite plugin-react conflict | Pin `@vitejs/plugin-react@^4.7.0` (not @latest — requires Vite 8) |
| semgrep breaks deps | Install via `pipx`, never `pip` into app virtualenv |

## Key File Locations

**Config**: `backend/app/core/config.py`, `frontend/vite.config.ts`, `frontend/components.json`, `frontend/src/index.css` (@theme)
**Design**: `design/DESIGN.md` (tokens), `design/PRD.md` (4 screens), `design/stitch/v3-evolved/` (latest renders)
**Migrations**: `backend/alembic/versions/`
**Solutions**: `docs/solutions/` — documented patterns, bug fixes, past mistakes. **Check before implementing anything similar.**

## Implementation Status

**Complete**: Lead tutor + Socratic chat, multi-provider AI, file upload/processing, chat streaming, subject CRUD, session lifecycle (start/pause/resume/stop), dashboard summary API, dashboard UI (HeroMetrics, SubjectList, ContributionHeatmap, CTA), focus timer (Web Worker), Tailwind v4 foundation, auth forms restyled, Stitch v3 evolved designs, concept extraction pipeline (Claude Structured Outputs + parallel chunks), Subject Detail page (MasteryRing, ConceptCard, ExtractionTrigger), per-subject mastery in dashboard, empty extraction UX, content deletion cascade warning, security hardening (session 10), full-text search (tsvector + GIN index), auth hardening (httpOnly cookies only, refresh token rotation with Redis families, no tokens in response body).

**Phase 2**: COMPLETE (PR #26 + PR #29 + PR #31 + PR #51 + PR #53 + PR #54 + PR #55 + PR #58 merged). Follow-up todos (009-049) resolved across sessions 10-15.
**Phase 3 (next)**: Chat restyle (MUI → Tailwind), react-markdown integration, MUI removal.
**Phase 4**: Practice generation, attempt tracking, AI grading, real Active Focus.
**Phase 5**: SM-2 scheduling, analytics page, recommendation engine, retention curves.
**Phase 6**: Monetization (Stripe, usage tracking, tier enforcement). Strategy at `docs/brainstorms/2026-03-14-monetization-strategy-brainstorm.md`.

See [docs/technical/IMPLEMENTATION_STATUS.md](docs/technical/IMPLEMENTATION_STATUS.md) for detailed status.

## Documentation Index

| Doc | What |
|-----|------|
| [DEVELOPMENT.md](DEVELOPMENT.md) | Local setup guide |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Production deploy |
| [design/DESIGN.md](design/DESIGN.md) | Design system (canonical) |
| [design/PRD.md](design/PRD.md) | Analytics Pro PRD (4 screens) |
| [docs/technical/ARCHITECTURE.md](docs/technical/ARCHITECTURE.md) | System architecture |
| [docs/technical/IMPLEMENTATION_STATUS.md](docs/technical/IMPLEMENTATION_STATUS.md) | Current status |
| [docs/direction/NEW_DIRECTION_2025.md](docs/direction/NEW_DIRECTION_2025.md) | Strategic direction |
| [docs/brainstorms/2026-03-13-mvp-frontend-brainstorm.md](docs/brainstorms/2026-03-13-mvp-frontend-brainstorm.md) | Stack decisions, tool audit |
| [docs/plans/2026-03-14-001-feat-full-product-build-phases-neg1-0-1-plan.md](docs/plans/2026-03-14-001-feat-full-product-build-phases-neg1-0-1-plan.md) | Full build plan |
| [docs/solutions/](docs/solutions/) | Bug fixes, patterns, past mistakes |
| [docs/README.md](docs/README.md) | Full docs index |

## Best Practices

1. Check `docs/solutions/` before implementing anything similar to past work
2. New UI: Tailwind + shadcn/ui only (not MUI). Use design tokens from `@theme`.
3. Run tests before pushing. Read FULL CI log on failure — fix root cause, not symptoms one by one.
4. Backend: type hints, PEP 8, ruff. Frontend: strict TypeScript, ESLint, functional components.
5. Conventional commits. Feature branches from main.

---
**Last Updated**: March 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/belumume) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
