## crypto-portfolio-tracker-oss

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Backend (Python 3.12, FastAPI, SQLAlchemy, SQLite)

```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
SECRETS_KEY=$(python -c 'from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())') \
SESSION_COOKIE_SECURE=false \
uvicorn app.main:app --reload          # http://localhost:8000
```

Health check: `GET /api/health`. Interactive docs at `/docs` when `ENABLE_API_DOCS=1`.

### Frontend (React 18 + TypeScript + Vite)

```bash
cd frontend
npm install
npm run dev                             # http://localhost:5173 (proxies /api → :8000)
npm run build                           # tsc -b && vite build → frontend/dist
```

### Full stack (Docker Compose)

```bash
cp .env.example .env                    # fill in SECRETS_KEY + provider keys
docker compose up --build -d            # SPA on :80, API on :8000
```

### Tests / Lint

There is no test suite, linter config, or formatter config in the repo. Don't claim changes are "tested" unless you actually exercised them against the running backend/frontend. The frontend `npm run build` runs `tsc -b` first, so a green build is the type-check signal.

## Architecture

### Two-process layout

- **`backend/`** FastAPI app exposing `/api/*`. Single Uvicorn process, SQLite file at `$PORTFOLIO_DB_PATH` (default `<repo>/data/portfolio.db`). Set `DATABASE_URL` (e.g. `postgresql+psycopg://…`) to move to Postgres; engine selection is in `backend/app/db.py`.
- **`frontend/`** Vite SPA. In dev, Vite proxies `/api` to `:8000`. In prod, Nginx serves the built SPA and proxies `/api/` to the `backend` container (`frontend/nginx.conf`).

### Auth model

Local multi-user auth. Per-user, opaque session tokens (the cookie holds the raw token; only its SHA-256 is stored in the `sessions` table) set as an `HttpOnly` cookie (`portfolio_session`). Passwords use stdlib `hashlib.scrypt` (`backend/app/auth.py`). **Signup logs the user in immediately** — there is no email-confirmation or password-reset flow (a forgotten password is reset in the DB). Every authenticated endpoint depends on `current_user`, which resolves the cookie → session row → user row. On the frontend, `AuthContext` (`frontend/src/auth/AuthContext.tsx`) owns login/signup/logout and calls `setCacheScope(user.id)` so the `useApi` cache is namespaced per user; on logout `clearApiCache()` wipes memory + localStorage so a previous user's data can't leak.

### Source taxonomy (onchain / exchange / custom)

The whole app keys off `account.source`:

- `onchain` — EVM or Solana/Sui/Cosmos wallet. Dispatch in `services/sync.py::_sync_onchain`. EVM uses DeBank's `/total_balance` (wallet tokens across chains) **plus** `/complex_app_list` (DeFi positions) summed together — `total_balance` alone misses lending/LP/staking positions. Solana/Sui/Cosmos use CoinStats.
- `exchange` — centralized or perp DEX. Exchange is resolved by `_infer_exchange`: prefer the explicit `exchange` field on the `cex_credentials` row, fall back to address-prefix sniffing for legacy rows. Credentials live per-account in the DB, not in `.env`.
- `custom` — manual entry; no sync, balance edited through the UI.

An old taxonomy (`chain` / `cex` / `perp` / `manual`) is rewritten in place by `db.py::_migrate_source_values` on every boot — idempotent, keep it that way.

### Credentials split

- **Global env** (`.env` at repo root): `SECRETS_KEY` (encrypts stored credentials), `DEBANK_ACCESS_KEY`, `COINSTATS_API_KEY`, `COINMARKETCAP_API_KEY`. Loaded by `main.py` via `load_dotenv(find_dotenv(usecwd=True))`; Docker Compose injects the same vars via `env_file`.
- **Per-account** (DB `cex_credentials` table): `api_key`, `api_secret`, `passphrase`, `wallet_address`, `private_key`. Entered through the UI; the secret columns are encrypted at rest (`crypto.py` / `EncryptedString`), and the `credentials` router only exposes `has_*` booleans.

### Snapshots & history

Each account has at most one `account_snapshots` row (primary key = `account_id`) holding the last sync's balance, provider, and `holdings` JSON. Every sync also appends to `account_snapshot_history`. Aggregate history lives in `total_snapshots` — one row per user per sync call (written in `services/sync.py`). The balance-history endpoint reads these for charts.

### Validation vs sync

`services/sync.py` has two paths:

- `sync_account` / `sync_user_accounts` — live fetch, persists snapshot, returns `SyncResult` with `ok` / `skipped` / `error`. Partial failures are tolerated (a missing `/complex_app_list` doesn't fail the whole sync). A lightweight in-process throttle (`ratelimit.check_sync_allowed`) guards against accidental spam of the paid provider APIs.
- `validate_account` — dry-run fetch used by the accounts router on PATCH. Accepts a `pending_cred` so in-memory edits to credentials can be validated before commit. Raises `ValidationFailed`; the router must `db.rollback()` on failure.

### Frontend data layer

`src/api.ts` is the single typed API client (`fetch` with `credentials: "include"`). All hooks go through `src/hooks/useApi.ts`, which implements stale-while-revalidate with a two-tier cache (in-memory `Map` + `localStorage`). `key` must be passed to opt into caching; the cache is scoped per user via `setCacheScope`. Route structure is a flat `Routes` tree in `App.tsx` with a `RequireAuth` wrapper around the authenticated shell.

## Gotchas

- **No Alembic / migrations framework.** Schema changes go through `Base.metadata.create_all` at startup. For data migrations, follow the `_migrate_source_values` pattern: idempotent SQL run inside `init_db()`.
- **DeBank balance = tokens + positions.** Never use `total_usd_value` from `/total_balance` alone — always add the summed `usd` from positions produced by `_build_evm_holdings`.
- **Session cookie is Secure-by-default.** Set `SESSION_COOKIE_SECURE=false` for local HTTP dev (docker-compose already does); flip it back behind HTTPS.
- **`SECRETS_KEY` is required.** It encrypts CEX keys / wallet private keys at rest. Losing it means losing every stored credential. The app fails fast on boot if it's missing or malformed.
- **i18n must stay mirrored.** `frontend/src/i18n/zh.ts` is typed as `TranslationDict` (= `typeof en`), so `en.ts` and `zh.ts` must have exactly the same keys or the build fails.
- **Mock/truncated addresses.** `_is_mock_address` treats any address containing `…` or `...` as unsync-able (skipped, not errored).

---
> Source: [your-quantguy/crypto-portfolio-tracker-oss](https://github.com/your-quantguy/crypto-portfolio-tracker-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
