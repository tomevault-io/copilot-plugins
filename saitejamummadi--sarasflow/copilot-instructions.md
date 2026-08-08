## sarasflow

> You are working in SARAS, an open-source AI video platform for education: topic in → script → narration → images → assembled video with word-timed captions out. FastAPI backend, Next.js 14 frontend. A human probably pointed you here to set the project up, extend it, or customize a fork. This file tells you how the house works.

# Agent Guide — SARAS

You are working in SARAS, an open-source AI video platform for education: topic in → script → narration → images → assembled video with word-timed captions out. FastAPI backend, Next.js 14 frontend. A human probably pointed you here to set the project up, extend it, or customize a fork. This file tells you how the house works.

## Setup and run

```bash
# Backend (Python 3.11+)
cd backend
python3 -m venv .backenv
source .backenv/bin/activate
pip install -r requirements.txt
cp .env.example .env          # human fills in keys; see "Environment" below
fastapi run app.py            # http://localhost:8000, docs at /docs
                              # use `fastapi dev app.py` for auto-reload

# Frontend (Node 18.17+)
cd frontend
npm install
cp .env.local.example .env.local
npm run dev                   # http://localhost:3000
```

First backend startup auto-creates the database schema (`scripts/database.py`) and downloads caption fonts into `uploads/`. No migration commands exist or are needed.

## Verify your work

```bash
cd backend && python -m pytest tests/ -q
```

The suite (270+ tests) is hermetic: it mocks the database and every provider, injects its own credentials in `tests/conftest.py`, and needs no API keys and no network. **Keep it that way** — never write a test that reads real `.env` values or hits a real service, and never hardcode a real credential, invite code, or password anywhere, including test fallbacks.

For frontend changes, `npm run build` catches type and build errors; there is no frontend test suite.

## Environment

- All secrets live in `backend/.env` and `frontend/.env.local` (gitignored). The committed `.env.example` / `.env.local.example` files document every variable. Never commit, print, or copy real values from the env files into code, tests, docs, or command output.
- `ADMIN_PASSWORD` has no default — the backend intentionally refuses to start without it. Do not "fix" this by adding a fallback.
- `DATABASE_URL` must be a Supabase **pooler** URL (`aws-*.pooler.supabase.com`). The direct `db.*.supabase.co` host is IPv6-only and fails to connect from most networks.

## Hard constraints — do not "upgrade" these

- **Stripe SDK is currently pinned `<15.0.0`** (`requirements.txt`). At the time of pinning, v15 broke `Session.retrieve()`, which breaks checkout verification. Don't lift the pin as a drive-by; only upgrade deliberately, after verifying the checkout flow against the newer SDK.
- **Next.js stays on 14.** v15 requires Node 20; this project supports Node 18.17+.
- **Prices come from `backend/scripts/config/pricing.json` and credit packages from `credit_packages.json` — server-side only.** Never accept a price or credit amount from the client. Every generation must pass through `_deduct_credits()` in `app.py`.

## Repo map

| Path | What it is |
|---|---|
| `backend/app.py` | All API endpoints in one file — this is deliberate; add new endpoints here |
| `backend/services/` | Provider clients: Gemini, OpenAI, Runware, R2, Whisper, email |
| `backend/scripts/*.md` | Prompts, stored as files — the content brain of the product |
| `backend/scripts/longscriptprompt.md` | Defines the 12 narrative styles (TEACH, HOT TAKE, STORY TIME, MYTH BUST, POV, LIST DROP, CONFESSION, STAT SHOCK, CASE STUDY, MYSTERY, DOCUMENTARY, PROBLEM-SOLUTION) — injected via `{NARRATIVE_STYLE}` |
| `backend/scripts/config/` | `pricing.json`, `credit_packages.json`, `runware_models.json` |
| `backend/scripts/database.py` | Schema + `create_tables()` |
| `backend/scripts/karaoke_render.py`, `caption_styles.py` | Caption rendering and presets |
| `backend/tests/` | Hermetic pytest suite |
| `frontend/types/index.ts` | `AGENT_CONFIGS`, `VOICE_MAP`, `VISUAL_STYLES` — the registry of what exists |
| `frontend/lib/` | `api.ts` (all backend calls), `auth.tsx`, `theme.tsx` |

## Common extension tasks

**Add a language:** copy the prompt folder pattern — short-form prompts live in `backend/scripts/` (English) and `backend/scripts/telugu/`; long-form under `backend/scripts/long/<language>/`. Translate the four prompt files per agent, then extend the language mapping in `getAgentConfig()` in `frontend/types/index.ts`.

**Add a narrative style:** define its block in `backend/scripts/longscriptprompt.md` (hook pairing, structure, pacing — follow the existing style blocks) and add the name to the `Narrative Style` list in `frontend/app/dashboard/generate/page.tsx`. The frontend sends it as `narrative_style`; the backend substitutes it into the script prompt (defaults to `TEACH`).

**Add a content agent (legacy path):** the older agent system (`_AGENT_CONFIGS` / `AgentType` in `frontend/types/index.ts`, four prompt files each) still exists and routes prompt selection per agent/language/format. Narrative styles are the current design; touch agents only when a task explicitly involves them.

**Add a caption preset:** add an entry to `CAPTION_PRESETS` in `backend/scripts/caption_styles.py`; fonts referenced there auto-download via `font_manager.py`.

**Add a visual style:** add the style entry to `VISUAL_STYLES` in `frontend/types/index.ts` and create the matching image-prompt file.

## Conventions

- Frontend: Tailwind + Radix primitives only — no shadcn, no component libraries. All colors go through the CSS variables in `globals.css` (`--background`, `--card`, `--border`, …) so dark/light theming keeps working. Primary purple is `#7c3aed`.
- Backend: follow the existing patterns in `app.py` — psycopg2 with explicit `conn.close()` in `finally`, HTTPException for errors, JWT auth via the existing helpers.
- Every generation endpoint costs the operator real API money. If you add one, price it in `pricing.json` and meter it with `_deduct_credits()`.

---
> Source: [SaiTejaMummadi/sarasflow](https://github.com/SaiTejaMummadi/sarasflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
