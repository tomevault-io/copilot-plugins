## privatescribe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

PrivateScribe.ai — a private AI scribe that transcribes audio with **Faster-Whisper** locally and reformats the transcript into Markdown via a **local Ollama** model (default `llama3.2`). Two apps in one repo:

- `backend/` — Flask + SQLAlchemy + SQLite, exposes a JWT-protected JSON API on `http://127.0.0.1:5000`.
- `frontend/` — Vite + React 19 + TypeScript + Tailwind v4 + shadcn/ui (Radix), served on `http://127.0.0.1:3000`.

CORS is hardcoded to `http://localhost:3000` in `backend/app.py`; the frontend hardcodes `http://127.0.0.1:5000` as the API base in fetch calls. If either changes, both sides need updating.

## Common commands

### Backend (run from `backend/`)
```bash
source venv/bin/activate
flask run                  # serves on :5000
flask db upgrade           # apply Alembic migrations
flask db migrate -m "..."  # generate migration after model change
flask create-admin         # CLI command (defined in app.py) to create an admin user
```
The SQLite DB lives at `backend/instance/privatescribe.db`. `db.create_all()` runs at import time, so the schema is also created implicitly on first server boot.

### Frontend (run from `frontend/`)
```bash
npm install
npm run dev      # vite dev server on :3000
npm run build    # tsc -b not wired — `vite build` only
npm run lint     # eslint .
npm run preview
```

### Ollama (must be running for `/api/getMarkdown`)
```bash
ollama serve
ollama pull llama3.2
```

## Architecture

### Backend — single-file Flask app

`backend/app.py` is monolithic (~1100 lines) and contains models, routes, and the CLI command together. When adding endpoints or fields, edit this file directly rather than splitting it.

**Models** (all use `String(36)` UUID primary keys except the `note_participants`/`user_participants` join tables):
- `User` — auth subject. Owns `notes`, `templates`, `participants` with `cascade='all, delete-orphan'`.
- `Template` — Markdown skeleton with `[bracketed instructions]` that drive LLM filling. Has `version`, `is_deleted`, `is_deleted_timestamp` (soft delete pattern shared with Note).
- `Note` — has `note_content_raw` (Whisper output) and `note_content_markdown` (Ollama-formatted output) side-by-side, plus optional `template_id` and many-to-many `participants`.
- `Participant` — author-scoped contacts.

**Soft delete pattern**: notes and templates use `is_deleted` + `is_deleted_timestamp` instead of row deletion; there are separate `/delete`, `/restore`, and `/delete-permanently` endpoints. Honor this when adding queries — most read endpoints filter `is_deleted=False`.

**Two-step note creation flow**:
1. `POST /api/transcribe` — multipart audio file → `faster-whisper` (`base` model, CPU, int8) → raw text. Non-WAV uploads are converted via `pydub` (ffmpeg required on the host).
2. `POST /api/getMarkdown` — raw text + `note_details` (with `template_id`) → Ollama `llama3.2` chat with a system prompt that instructs the model to fill `[instructions]` inside the template verbatim, leaving non-bracketed text untouched. Temperature is 0.2.
3. `POST /api/notes` then persists the result.

The Ollama system prompt in `getMarkdown` is the contract for how templates work — bracketed instructions are filled, everything else is preserved literally. Edits to that prompt change template behavior across the app.

**Auth**: `flask_jwt_extended` issues access tokens (1h) and refresh tokens. Most routes are guarded by `@jwt_required()`; the frontend stores tokens in `localStorage` and validates via `GET /api/validateToken`.

**Heads-up — `requirements.txt` is incomplete.** It omits several runtime deps the app imports: `flask_sqlalchemy`, `flask_jwt_extended`, `flask_cors`, `flask_migrate`, `python-dotenv`, `pydub`. A fresh install from `requirements.txt` alone will not boot. The committed `backend/venv/` has them installed; if recreating an env, install the missing ones explicitly.

**`backend/.env`** must define `JWT_SECRET_KEY`. Loaded via `python-dotenv`.

### Frontend — Vite SPA

- Routing is centralized in `src/main.tsx` with `react-router` v7. Protected routes are wrapped in `<RequireAuth>` (`src/components/auth/RequireAuth.tsx`), which calls `/api/validateToken` on mount and redirects to `/login` on failure.
- Auth state lives in `src/context/auth-context.tsx` (`useAuth` hook). Tokens and user object are mirrored to `localStorage`.
- `@/` is aliased to `src/` (see `vite.config.ts` and `tsconfig.json`).
- Two parallel UI styles coexist:
  - `src/components/ui/*` — shadcn/ui Radix primitives (added via `components.json`).
  - `src/components/neo/*` — a custom "neobrutalist" design system used by the marketing/home/roadmap pages (`neobrutal-home.tsx`, `neo-*` components).
- `vercel.json` rewrites all paths to `index.html` for SPA routing on Vercel.
- `frontend/.env` holds `VITE_EMAILJS_SERVICE` / `VITE_EMAILJS_USER` for the contact form (EmailJS).

### Migrations

Alembic is configured under `backend/migrations/` but `migrations/versions/` is empty — schema currently relies on `db.create_all()`. If you change a model, generate a migration with `flask db migrate` rather than relying on `create_all` for existing databases.

## Conventions worth knowing

- Both `.gitignore`s list `CLAUDE.md`, so this file is intentionally untracked.
- The frontend hardcodes API URLs throughout (`http://127.0.0.1:5000/...`). There is no central API client; if you need to change the host, grep for the literal.
- IDs are stringy UUIDs everywhere except the legacy `note_participants` join (`note_id` is `Integer` while `Note.id` is `String(36)` — pre-existing inconsistency, don't "fix" without a migration).

---
> Source: [secondpathstudio/privatescribe](https://github.com/secondpathstudio/privatescribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
