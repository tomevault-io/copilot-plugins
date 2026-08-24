## muktasabat

> - **API**: Python **FastAPI** + SQLAlchemy 2 (`api/`) — served with **Uvicorn**

# Muktasabat (PropOS)

## Stack (repo root)

- **API**: Python **FastAPI** + SQLAlchemy 2 (`api/`) — served with **Uvicorn**
- **Web**: **Next.js 14** (`web/`)
- **Database**: **PostgreSQL** (Docker) or **SQLite** for quick local API runs

The legacy **Flask** UI was removed; this tree is **FastAPI + Next.js** only.

## Local development

From the repository root:

```bash
cd web && npm install
```

**Terminal 1 — FastAPI** (Postgres example; or omit `DATABASE_URL` to use default SQLite `sqlite:///./muktasbat.db`)

```bash
cd /path/to/muktasabat
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
DATABASE_URL=postgresql://muktasbat:muktasbat_secret@127.0.0.1:5432/muktasbat \
  .venv/bin/uvicorn api.main:app --reload --host 127.0.0.1 --port 8000
```

Start Postgres only: `docker compose up -d db`

**Terminal 2 — Next.js** (point at your API)

```bash
cd web && API_URL=http://127.0.0.1:8000 npm run dev
```

Open **http://localhost:3000**

### Default administrator account

Unless `DISABLE_BOOTSTRAP_ADMIN` is set, the API ensures **at least one user with role `admin`** on startup:

- **Empty database**: creates the account below.
- **Users exist but none are admins**: promotes a user named `admin` to role `admin`, or creates `admin` / `admin123` (using `bootstrap-admin@muktasbat.app` if `admin@muktasbat.app` is already taken).

| Field | Value |
|--------|--------|
| **Username** | `admin` |
| **Password** | `admin123` |

Set `DISABLE_BOOTSTRAP_ADMIN=1` once you manage admins yourself, and **change this password** before any production use. New accounts from **POST** `/api/v1/auth/register` get role `viewer` ([API docs](http://localhost:8000/api/docs)).

### Password reset

- **Web**: `/[locale]/forgot-password` and `/[locale]/reset-password?token=…` (linked from the login screen).
- **API**: `POST /api/v1/auth/forgot-password` `{ "email" }`, `POST /api/v1/auth/reset-password` `{ "token", "new_password" }`.
- **Email** is not sent yet; for local testing set **`RESET_PASSWORD_DEBUG=true`** in the API environment so the forgot-password response can include **`debug_reset_url`** (open it in the browser, with your locale prefix, e.g. `/en/reset-password?token=…`).
- **Production**: plug in SMTP (or a provider), send the reset link by email, and keep **`RESET_PASSWORD_DEBUG` off**. If the `users` table was created before reset columns existed, use a fresh DB or run a migration so `password_reset_token` / `password_reset_expires` exist.

## Docker — full stack (Postgres + FastAPI + Next.js)

```bash
docker compose up --build
```

- Next.js: **http://localhost:3001** (compose default; inside the container it is still port 3000). Set `WEB_PORT=3000` in `.env` if you want the UI on port 3000.
- FastAPI docs: **http://localhost:8000/api/docs**

## Key paths

- `api/` — FastAPI application
- `web/` — Next.js application
- `docker-compose.yml` — `db`, `api`, `web`
- `Dockerfile` — API image
- Architecture: `PropOS_Architecture_Diagram.html`

## Rules

- Never commit `.env` or production secrets.
- `SECRET_KEY` must be set for JWT signing in non-dev environments.

---
> Source: [ElafNawaf/muktasabat](https://github.com/ElafNawaf/muktasabat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
