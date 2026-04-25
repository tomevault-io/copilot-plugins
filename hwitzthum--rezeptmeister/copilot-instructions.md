## rezeptmeister

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**Rezeptmeister** is a BYOK (Bring Your Own Key) AI-powered recipe management web app for the Swiss market. The full feature spec lives in `SPEC.md`. The implementation roadmap lives in `tasks/todo.md` (18 phases, each ending with Playwright E2E tests).

**Language:** All UI text, DB enums, error messages, and API responses must be in **German (Swiss standard)** — use "ss" not "ß", use Swiss units (dl, EL, TL, KL, °C) throughout.

---

## Architecture

Two separate runtimes that never import each other:

```
frontend/   → Next.js 15 (App Router, TypeScript) on port 3001
backend/    → FastAPI (Python, uv) on port 8000
db          → PostgreSQL + pgvector (Docker) on port 5432
uploads/    → Local filesystem (originals/ and thumbnails/) in dev
```

**Frontend** owns: auth (NextAuth.js), recipe CRUD API routes, image upload, shopping list, collections, meal plans, notes — all via Drizzle ORM directly to PostgreSQL.

**FastAPI backend** owns exclusively: AI pipeline — embeddings (Gemini Embedding 2), OCR, recipe suggestions, image generation, URL import, web search, nutrition calculation, semantic/hybrid search. It never handles auth; it receives the user's API key from the Next.js proxy on each request.

**Key architectural rule:** Next.js API routes proxy AI requests to FastAPI, injecting the user's decrypted API key server-side. The key is never sent to the browser.

---

## Common Commands

### Docker (start everything)
```bash
docker compose up -d          # Start PostgreSQL + pgvector + FastAPI
docker compose down           # Stop all services
docker compose logs backend   # FastAPI logs
docker compose logs db        # DB logs
```

### Frontend (Next.js)
```bash
cd frontend
npm install
npm run dev         # Dev server on http://localhost:3001
npm run build       # Production build (run this before merging — tsc alone misses next.js errors)
npm run lint        # ESLint
npx drizzle-kit generate   # Generate Drizzle migration from schema changes
npx drizzle-kit migrate    # Apply migrations to DB
npx drizzle-kit studio     # Drizzle Studio (visual DB browser)
```

### Backend (FastAPI + uv)
```bash
cd backend
uv sync                        # Install dependencies
uv run uvicorn app.main:app --reload --port 8000   # Dev server
uv run alembic upgrade head    # Apply DB migrations
uv run alembic revision --autogenerate -m "description"  # Generate migration
uv run pytest                  # All tests
uv run pytest tests/test_embeddings.py  # Single test file
uv run pytest -k "test_ocr"    # Single test by name
```

### E2E Tests (Playwright)
```bash
cd frontend
npx playwright install         # First-time setup
npx playwright test            # Run all E2E tests
npx playwright test tests/phase-2.spec.ts   # Single phase test file
npx playwright test --ui       # Interactive UI mode
npx playwright show-report     # View last test report
```

---

## Key Conventions

### Database
- **Drizzle ORM** is used on the Next.js side for all non-AI DB access. Schema is in `frontend/src/lib/db/schema.ts`.
- **SQLAlchemy + Alembic** is used on the FastAPI side. Models are in `backend/app/models/`.
- Both ORMs point to the same PostgreSQL instance — the Drizzle schema is the source of truth for table structure; Alembic migrations must stay in sync.
- `embedding` columns are `VECTOR(3072)` (Gemini Embedding 2 default). Use HNSW index, cosine distance (`<=>` operator).
- Drizzle does not natively support `VECTOR` type — use `customType` wrapper.

### Next.js 16 – Breaking Changes
- **`middleware.ts` is renamed to `proxy.ts`** (same API, just the filename). Export a named `proxy` function or a default export. `export const config` with `matcher` still works the same.
- Route Handlers: same as Next.js 15 (`route.ts` with `GET`, `POST`, etc.)
- Fonts: `next/font/google` works the same (variable + className approach)

### Authentication & Security
- NextAuth.js Credential Provider. JWT sessions, 24h timeout.
- Passwords: bcrypt (never plain).
- User `status` must be `'approved'` to log in — enforce in NextAuth `authorize` callback.
- API keys: AES-256 encrypted at rest using `ENCRYPTION_KEY` env var (32-byte hex). Decrypted only in server-side API route handlers, never returned to client (only masked placeholder shown).
- Rate limiting: `express-rate-limit` on every Next.js API route (100 req/15 min default), stricter on `/api/ai/*` and `/api/auth/*`.
- Validate all inputs with `zod` at API route entry points.

### AI / Embeddings (FastAPI)
- Embedding model: `gemini-embedding-2-preview`, 3072 dims.
- Use `task: search_query` prefix for query embeddings, `task: search_document` for document embeddings.
- Embedding creation is always async (Background Task) — never block the HTTP response.
- All AI endpoints receive the user's API key in the request body (forwarded by Next.js proxy from server-side session). FastAPI does NOT read from DB for API keys.

### Image Handling
- Upload: validate MIME type (`image/jpeg`, `image/png`), max 10 MB.
- Thumbnail: 300×300 WebP via `sharp` (Next.js side) stored in `uploads/thumbnails/`.
- Serve uploads via Next.js static route (dev) or S3 URL (prod).
- Image embeddings are created asynchronously by FastAPI after upload.

### Swiss Units
- All unit dropdowns, DB values, and conversions use Swiss standard: `g, kg, ml, dl, l, EL, TL, KL, Msp., Prise, Stk., Bund, Pkg., Scheibe, Dose, Becher, Pfd.`
- Conversion utilities live in `frontend/src/lib/units/`.
- Auto-convert on import: US cups → dl/g, °F → °C, oz → g, lb → kg.

### Design System
- **Colors:** Primary `#C24D2C` (Terrakotta), Secondary `#FFF8F0` (Cremeweis), Accent `#D4A843` (Gold), warm grays.
- **Dark mode:** Supported — darkened warm tones, toggled via `next-themes`.
- Use the `frontend-design` skill when creating new pages or UI components to ensure a distinctive, non-generic look.
- Mobile-first: bottom navigation bar on mobile (5 main items), sidebar on desktop.

### MANDATORY RULES — Testing & Completion (non-negotiable)

**1. Every feature must be tested before it is considered done.**
- Unit tests for pure functions and utilities
- Integration tests for API routes, DB queries, and auth logic
- Playwright E2E tests for all user-facing flows
- A feature is NOT done until all applicable test layers pass. "It runs" is not a definition of done.

**2. Every checkbox in `tasks/todo.md` for the current phase must be fully implemented before advancing.**
- Never skip a checklist item to move forward.
- If an item is blocked by a real dependency, document the blocker explicitly and resolve it before marking done.
- No phase-skipping, no partial implementations declared complete.

These rules exist because previous sessions skipped todo items and omitted test layers. Violations will be called out.

### Testing Strategy
- Each implementation phase gets a Playwright test file: `frontend/tests/phase-X.spec.ts`.
- Backend: Pytest integration tests that hit a real PostgreSQL instance (never SQLite mocks).
- Mark Pytest tests with `pytest.mark.skipif` on DB connection check so they skip gracefully when DB is unavailable.
- Playwright runs on port 3002. Port 3000 is occupied by open-webui (Ollama UI); Rezeptmeister dev server runs on port 3001.

---

## Environment Variables

```bash
# frontend/.env.local
DATABASE_URL=postgresql://rezeptmeister:localdev@localhost:5432/rezeptmeister
NEXTAUTH_SECRET=<random-secret>
NEXTAUTH_URL=http://localhost:3001
BACKEND_URL=http://localhost:8000
UPLOAD_DIR=./uploads
ENCRYPTION_KEY=<32-byte-hex-key>

# backend/.env (or injected via docker-compose)
DATABASE_URL=postgresql://rezeptmeister:localdev@db:5432/rezeptmeister
UPLOAD_DIR=/app/uploads
```

User-provided AI API keys (Gemini, Claude, OpenAI) are stored encrypted in the DB — never in env files.

---

## Project Structure (top-level)

```
SPEC.md              # Full feature specification — source of truth
CLAUDE.md            # This file
tasks/todo.md        # 18-phase implementation plan with checkboxes
docker-compose.yml   # PostgreSQL/pgvector + FastAPI
db/init.sql          # Schema DDL + pgvector extension
db/seed.sql          # Swiss test recipes
frontend/            # Next.js app
backend/             # FastAPI AI pipeline
uploads/             # Local image storage (dev only, gitignored)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hwitzthum) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
