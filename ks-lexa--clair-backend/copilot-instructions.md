## clair-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CLAIR Backend is a FastAPI-based AI contract-analysis system (Korean-language app, port 8000). It handles contract upload, AI-powered analysis, chat-based Q&A, notifications, and public share links. MySQL + SQLAlchemy ORM, JWT auth with social login (Google/Naver/Kakao), and email-based flows (signup verification, password reset).

**The AI itself lives in a separate service (`clair-ai`, default `http://localhost:8001`).** This backend never calls Gemini/LLMs directly — it POSTs to clair-ai's `/analyze` and `/qa` endpoints. `GEMINI_API_KEY` in config is vestigial; `AI_SERVICE_URL` is what matters.

## Commands

```bash
# Setup
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS clair_db DEFAULT CHARACTER SET utf8mb4;"
cp .env.example .env          # then fill DB_PASSWORD, SECRET_KEY, AI_SERVICE_URL, etc.

# Run dev server
source venv/bin/activate
uvicorn app.main:app --reload --port 8000
# Swagger /docs · ReDoc /redoc · Health /health

# DB migrations (Alembic)
alembic upgrade head                          # apply (run once on fresh DB)
alembic revision --autogenerate -m "설명"      # generate migration from model changes
alembic downgrade -1                          # roll back one
```

There is **no test runner configured** — `tests/` exists but is empty, and `pyproject.toml` declares no pytest config. Verify changes by running the server and exercising endpoints (the codebase convention; see git history / PR descriptions).

> macOS networking gotcha: `localhost` can resolve to IPv6 (`::1`) while uvicorn binds IPv4, breaking backend↔clair-ai calls. Use `127.0.0.1` in `AI_SERVICE_URL` and CORS origins.

## Architecture

Layered: **API (`app/api/v1/`) → Services (`app/services/`) → Models/DB (`app/models/`, `app/db/`)**. Route handlers stay thin — no business logic; all logic lives in services, which receive a SQLAlchemy `db` session and the current `User`. Pydantic v2 DTOs in `app/schemas/` are kept separate from ORM models.

Five routers, all mounted in [app/main.py](app/main.py): `auth`, `contracts`, `chat`, `notifications`, `shares`.

### Core singletons & dependencies
- [app/core/config.py](app/core/config.py) — `settings` singleton (Pydantic Settings from `.env`). Single source of truth for all config; has computed properties like `database_url`, `*_list`, `*_bytes`.
- [app/core/security.py](app/core/security.py) — JWT create/decode, password hashing, and the `get_current_user` dependency (`OAuth2PasswordBearer`). Access token 60 min, refresh 7 days.
- [app/db/session.py](app/db/session.py) — `engine`, `SessionLocal`, `Base`, and the `get_db()` dependency injected into every DB-touching route.
- [app/db/init_db.py](app/db/init_db.py) — `init_db()` runs `Base.metadata.create_all()` on startup (lifespan in main.py). **Schema is managed by THREE mechanisms** — `create_all` (boots any missing table, so a forgotten migration can silently "work" locally), Alembic ([alembic/versions/](alembic/versions/), the source of truth), and hand-written SQL in [scripts/](scripts/) (`migrate_2026_05_*.sql` applied manually against shared/prod DBs) plus the original [database/schema.sql](database/schema.sql). For any schema change, always write the Alembic migration; the raw SQL scripts are a parallel manual track, not auto-applied.

### Contract analysis flow (async state machine)
Status: `UPLOADED → PENDING → PROCESSING → COMPLETED / FAILED` (`ContractStatus` enum).

`POST /contracts/{id}/analyze` returns **202 immediately**, then runs `analyze_contract_background(contract_id)` as a FastAPI `BackgroundTask`. That background task **opens its own `SessionLocal()`** (the request-scoped `db` is already closed by the time it runs) and walks the contract through PROCESSING → COMPLETED/FAILED. Frontend polls `GET /contracts/{id}/status` or `GET /contracts/{id}`.

**Critical gotcha:** BackgroundTask exceptions are swallowed (not surfaced to the request). If a contract is stuck in `PROCESSING`, the cause is usually an unhandled error in the background task or an AI-response schema mismatch (see below).

### AI integration (`app/integrations/`)
- [ai_client.py](app/integrations/ai_client.py) — `AIServiceClient` singleton `ai_client`. **The only place that talks to clair-ai.** Services import `ai_client`, never `httpx` or AI types directly. `analyze_contract()` sends a server-local file path (clair-ai reads the file itself); `answer_question()` sends serialized clause dicts as RAG context.
- [ai_models.py](app/integrations/ai_models.py) — Pydantic models for clair-ai responses. **When clair-ai adds a field, mirror it here.** A missing/changed field raises a validation error inside the background task that gets swallowed → contract stuck in PROCESSING.
- [mappers.py](app/integrations/mappers.py) — pure functions converting AI response models → ORM rows (`ContractClause`, `RiskClause`, `ComplianceResult`, `AnalysisResult`).

### Safety scoring ([app/services/scoring.py](app/services/scoring.py))
`compute_safety_score(risk_clauses)` starts at 100 and deducts `CATEGORY_WEIGHTS[risk_type] × severity × confidence`, floored at 25. `GET /contracts/{id}` returns `safety_score` (int) and `safety_score_detail` (dict). Computed in the backend, not by the AI.

### PDF reports ([app/services/pdf_service.py](app/services/pdf_service.py))
WeasyPrint (HTML→PDF) renders a Jinja2 template ([app/templates/pdf/contract_report.html](app/templates/pdf/contract_report.html)) for `GET /contracts/{id}/download/pdf` (COMPLETED contracts only). Korean display labels (risk levels, `key_info` keys) are mapped at render time via Jinja filters — the underlying data/API responses are never mutated for display.
> macOS gotcha encoded at the top of pdf_service.py: WeasyPrint dlopens pango/glib, but SIP strips `DYLD_*` from child procs. The module sets `DYLD_FALLBACK_LIBRARY_PATH` to the Homebrew lib dir **before** importing weasyprint — keep that ordering.

### File storage & static serving
Uploaded contracts go under `UPLOAD_DIR` by date (`YYYY/MM/DD/`); `stored_filename` is UUID-based, `original_filename` is preserved. Multi-image uploads are merged into a single PDF (`_merge_images_to_pdf`, Pillow). Allowed: `.pdf,.png,.jpg,.jpeg,.txt,.docx`, max 20 MB.

Profile images are stored separately under `UPLOAD_DIR/profile_images/` and are the **only** thing served statically — main.py mounts just that subdir (not all of `UPLOAD_DIR`) so contract files are never web-exposed. `User.profile_image_path` holds a relative path; `profile_image_url()` turns it into an absolute URL via `BACKEND_BASE_URL`. Allowed: `.png,.jpg,.jpeg,.webp`, max 5 MB.

### Auth & email flows ([app/services/auth_service.py](app/services/auth_service.py))
- **Signup requires prior email verification**: `email-verification/request` sends a 6-digit code; `email-verification/confirm` marks an `EmailVerification` row verified; `signup` consumes (deletes) that row. No verified row → signup 400s.
- **Social login** (Google/Naver/Kakao): OAuth2 code flow. Callback exchanges code for tokens via `httpx`, then `_social_login()` upserts `SocialAccount` + `User` and issues app JWTs. Social users have `password_hash=None` (and thus can't change password).
- **Password reset**: emailed token (`PasswordResetToken`), verified then consumed. Endpoints intentionally return identical messages regardless of whether the email exists.
- Email sending is in [app/integrations/email_client.py](app/integrations/email_client.py) (fastapi-mail / SMTP).

### Contract sharing ([app/services/share_service.py](app/services/share_service.py))
Owners create tokenized share links (`ContractShare`); the public `/api/v1/shares/{token}` router lets unauthenticated users view results. Share access uses short-lived per-link access tokens distinct from user JWTs.

## Models
`User`, `SocialAccount`, `Contract`, `ContractClause`, `AnalysisResult`, `RiskClause`, `ComplianceResult` (in [analysis.py](app/models/analysis.py)), `ChatSession`, `ChatMessage`, `Notification`, `ContractShare`, `PasswordResetToken`, `EmailVerification`. New models must be imported in [app/db/init_db.py](app/db/init_db.py) so `create_all` and Alembic autogenerate see them.

## Conventions
- User-facing strings, commit messages, and code comments are in **Korean** — match the surrounding style.
- Runtime is **Python 3.9** (venv). Do **not** use PEP 604 unions (`X | None`) as runtime annotations — they crash on 3.9. Files use `from __future__ import annotations` where modern syntax appears; keep that import if you add such syntax.

## Environment Variables (`.env`)
Required: `DB_HOST/PORT/USER/PASSWORD/NAME`, `SECRET_KEY`, `UPLOAD_DIR`, `AI_SERVICE_URL` (+ `AI_SERVICE_TIMEOUT`), `CORS_ORIGINS`, `BACKEND_BASE_URL`, `FRONTEND_BASE_URL`.
Optional/feature: SMTP (`SMTP_HOST/PORT/USER/PASSWORD`, `MAIL_FROM`) for email flows; social login (`{GOOGLE,NAVER,KAKAO}_CLIENT_ID/SECRET/REDIRECT_URI`) — leave empty to disable. See [app/core/config.py](app/core/config.py) for the full list and defaults.

---
> Source: [KS-LEXA/clair-backend](https://github.com/KS-LEXA/clair-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
