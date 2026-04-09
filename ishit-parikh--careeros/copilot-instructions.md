## careeros

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the dev server
uvicorn app.main:app --reload

# Run tests (once tests exist)
pytest --cov=app --cov-report=term-missing

# Run a single test
pytest tests/path/to/test_file.py::test_function_name -v

# Generate a Fernet key (needed for AES_KEY env var)
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

## Environment Setup

Copy `.env.example` to `.env` and fill in all values. The app will refuse to start if `MONGO_URI`, `JWT_SECRET`, or `AES_KEY` are missing or if `AES_KEY` is not a valid Fernet key (`validate_settings()` runs at startup).

## Architecture

**FastAPI + MongoDB Atlas (Motor async driver).** The app is a backend-only API; CORS is pre-configured for `localhost:5173` (Vite frontend).

### Module layout

| Module | Role |
|--------|------|
| `app/config.py` | Frozen `Settings` dataclass loaded from env; `validate_settings()` called at lifespan startup |
| `app/db.py` | Motor connection lifecycle; creates all MongoDB indexes on startup |
| `app/security.py` | Crypto primitives: bcrypt passwords, Fernet AES encryption/decryption, JWT creation/decode |
| `app/security_layer.py` | IDS middleware logic: in-memory rate limiting, brute-force IP locking, audit log writes to MongoDB |
| `app/main.py` | App factory: mounts CORS, IDS middleware, all routers, exception handlers, `/health` endpoint |
| `app/schemas.py` | All Pydantic request/response models |
| `app/market.py` | APScheduler background job that fetches Adzuna jobs, GitHub trending repos, HN hiring threads, and Stack Overflow survey data; refreshes on a timer |
| `app/analysis.py` | Resume storage (PDF/DOCX → encrypted text in MongoDB), OpenAI-powered skill-gap and market analysis, dashboard assembly |

### Routes

| Prefix | File | Key endpoints |
|--------|------|---------------|
| `/auth` | `routes/auth.py` | `POST /signup`, `POST /login`, `GET /me`, `PUT /profile` |
| `/analysis` | `routes/analysis.py` | `POST /upload` (resume), `POST /run` (trigger analysis), `GET /dashboard` |
| `/market` | `routes/market.py` | Market overview queries |
| `/admin` | `routes/admin.py` | Hidden from OpenAPI docs; requires admin JWT; audit logs, security alerts, IP flagging |

### Security design

- **Email privacy**: emails are never stored in plaintext. `hash_email()` (SHA-256) is used as the lookup key; `encrypt_email()` (Fernet) stores the recoverable value.
- **Passwords**: bcrypt.
- **JWTs**: HS256, 24-hour expiry for users; 2-hour expiry for admins. Admin tokens carry `"role": "admin"` in the payload.
- **IDS middleware** (`ids_middleware` in `main.py`): every request is checked against flagged IPs (MongoDB), locked IPs (in-memory + MongoDB), and a sliding-window rate limiter (100 req/60 s per IP). Login failures exceeding 5 in 60 s lock the IP for 15 minutes.
- **Resume files**: raw extracted text is encrypted with Fernet before storage in MongoDB `resumes` collection.

### MongoDB collections

`users`, `resumes`, `analysis_results`, `audit_logs`, `security_alerts`, `flagged_ips`, `market_jobs`, `market_github`, `market_hn`, `market_so`, `market_status`

### Market data pipeline

`market.py` runs an `APScheduler` job at startup. `refresh_all_market_data()` pulls from four sources for each tracked role: Adzuna (job listings), GitHub trending, HN "Who's Hiring" threads, and a bundled Stack Overflow 2024 survey JSON (`app/data/stack_overflow_survey_2024.json`). Each source writes to its own collection with a `source_key` unique index to prevent duplicates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ishit-Parikh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ishit-Parikh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
