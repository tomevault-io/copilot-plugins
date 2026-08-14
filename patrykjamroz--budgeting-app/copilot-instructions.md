## budgeting-app

> Personal budgeting app with a Django REST Framework backend and Next.js 15 frontend. SQLite in dev. JWT auth throughout.

# Budgeting App

Personal budgeting app with a Django REST Framework backend and Next.js 15 frontend. SQLite in dev. JWT auth throughout.

## Architecture

```
budgeting-app/
├── backend/      Django 5.1 + DRF, Python 3.13, SQLite
└── frontend/     Next.js 15, TypeScript, Tailwind, shadcn/ui
```

## Dev Setup

Two terminals required — backend and frontend run independently.

**Backend** (Terminal 1):
```bash
cd backend
source venv/bin/activate
python manage.py runserver 8100     # http://localhost:8100
```

**Frontend** (Terminal 2):
```bash
cd frontend
npm run dev                          # http://localhost:3100 (port set in package.json)
```

After migrations or model changes:
```bash
cd backend && source venv/bin/activate
python manage.py makemigrations
python manage.py migrate
```

URLs:
- Frontend: http://localhost:3100
- API: http://localhost:8100/api/wallets/
- Admin: http://localhost:8100/admin

Ports chosen (8100 backend / 3100 frontend) to avoid colliding with other local projects on 8000/3000. Backend port is a runserver arg; frontend port is baked into the `dev`/`start` scripts. Override either via `.env` (`NEXT_PUBLIC_API_URL`, `CORS_ALLOWED_ORIGINS`).

## Data Model

```
User
 ├── Wallet (many)           — balance is computed (initial_value + sum of transactions)
 │    └── Transaction (many) — positive = income, negative = expense
 ├── TransactionCategory (many) — per-user, not per-wallet
 └── UserTransactionTag (many)  — per-user, not per-wallet
```

All primary keys are UUIDs.

### Amount Convention

**Positive = income, negative = expense.** This is enforced everywhere — in models, serializers, and frontend display. Never use a separate `transaction_type` field.

### Categories & Tags

- User-scoped (shared across wallets), NOT wallet-scoped
- Default categories are auto-copied to each new user via Django signal in `wallets/signals.py`
- Category "delete" is a soft delete (`is_archived=True`); tags are hard deleted
- Both support `is_visible` toggle: hidden items are excluded from dropdowns but remain on existing transactions

### Wallet Balance

Balance is **never stored**. It is computed on every serialization:
```python
balance = initial_value + sum(transaction.amount for all transactions)
```

### Wallet Deletion

`DELETE /api/wallets/{wallet_id}/` is a soft delete — sets `is_archived=True`, transactions are preserved. The wallet list excludes archived wallets; the wallet detail endpoint still returns them (so a direct link to an archived wallet keeps working).

### Savings Goals

- Per-wallet financial planning targets (e.g., "Wedding €500 by May 25")
- System calculates required monthly savings rate across all active goals
- Status: active, completed, missed (missed if target_date < today)
- No allocation mechanics — goals are forecasts, not fund reservations

## API Overview

All endpoints require JWT auth (`Authorization: Bearer <token>`). Access tokens live 60 min, refresh tokens 30 days, with rotation on (`SIMPLE_JWT` in `backend/config/settings.py`).

```
GET/POST   /api/wallets/
GET/PATCH/DELETE  /api/wallets/{wallet_id}/            (DELETE = archive, see below)
GET/POST   /api/wallets/{wallet_id}/transactions/?month=M&year=Y   (or date_from=&date_to=, unpaginated)
GET/PATCH/DELETE  /api/wallets/{wallet_id}/transactions/{id}/
GET/PATCH/DELETE  /api/transactions/{id}/
GET        /api/wallets/{wallet_id}/export/            (CSV download; optional month=, year=, date_from=, date_to=)
GET/POST   /api/wallets/categories/
GET/PATCH/DELETE  /api/wallets/categories/{id}/
GET/POST   /api/wallets/tags/
GET/PATCH/DELETE  /api/wallets/tags/{id}/
GET/POST   /api/wallets/{wallet_id}/goals/
GET/PATCH/DELETE  /api/wallets/{wallet_id}/goals/{goal_id}/
GET        /api/wallets/{wallet_id}/goals/summary/?month=M&year=Y
POST       /api/wallets/{wallet_id}/import/parse/
POST       /api/wallets/{wallet_id}/import/categorize/   (AI category suggestions, optional)
POST       /api/wallets/{wallet_id}/import/execute/
GET/POST   /api/wallets/import-rules/                    (learned keyword→category rules)
DELETE     /api/wallets/import-rules/{id}/
POST       /api/token/           (login)
POST       /api/token/refresh/
# POST     /api/register/         ⚠️ NOT IMPLEMENTED — planned (Auth batch). Create users via `createsuperuser`.
```

## CSV Import

Two-step flow (plus an optional AI step):
1. **Parse** (`/import/parse/`) — upload CSV, get columns + sample rows + unique values + date-format detection (`date_format`, `date_format_ambiguous`, and per-column `date_formats: {col: {format, ambiguous}}`)
2. **Categorize** (`/import/categorize/`, optional) — AI suggests a category per **unique** row description for rows that lack a matching mapped category. Returns `{suggestions: [{key, signature, count, category_id, category_name}], usage_warning, quota_exceeded}`.
3. **Execute** (`/import/execute/`) — supply column mapping + amount config + optional filters + optional `date_format`. Optionally include `ai_categories` ({normalized_signature: category_id}) to apply the reviewed AI suggestions.

Business logic lives in `backend/wallets/services.py` (`GenericCSVImportService`).

**Duplicate detection:** same wallet + date + amount + note, compared against a **snapshot taken before the import loop**. Rows created by the current import are not in the snapshot, so a file legitimately containing N identical rows imports all N; re-running the same file still skips everything. Rows with no mapped note are stored (and deduped) as `"Imported transaction"`.

**Date formats:** execute accepts `date_format` = `auto` (default) | `DMY` | `MDY` | `YMD`. `auto` pre-scans the entire mapped date column: any value with a day-position >12 settles the order for the whole file, a 4-digit leading component means year-first, and a file that proves nothing (or contradicts itself) falls back to `DMY` (EU user). Parsing is then **strict** to the resolved order — a day-first file never silently re-reads a row as month-first. The only always-accepted exception is an unambiguous 4-digit year-first date (`2024-01-15`), since a leading 4-digit year cannot be a day or month. Two-digit years (`15.01.24`) follow the resolved order. Rows fitting neither raise a normal per-row import error.

**Category matching on import** is case-insensitive (`name__iexact`), so a CSV saying "groceries" reuses the existing "Groceries" instead of creating a near-duplicate.

**AI auto-categorization:** constrained to the user's existing visible categories (never invents new ones). Each row's signature is built from all descriptive columns **except the mapped date and amount** (so identical merchants dedup regardless of date/amount). Batched via `wallets.ai.categorize_signatures()`; quota exhaustion mid-import is non-fatal (remaining rows left uncategorized). When `ai_categories` is passed, execute does **not** auto-create categories from the mapped column (unmatched mapped values fall through to the AI suggestion); the legacy path (no `ai_categories`) still auto-creates.

**Learned category rules (`ImportCategoryRule`):** a user-taught `keyword → category` mapping. During import, any transaction whose signature contains the keyword (case-insensitive substring, longest keyword wins) is categorized **before** the LLM is called — so teaching a merchant once cascades to all similar rows and persists across future imports (free/instant thereafter). In the review step each suggestion carries an editable `keyword` (auto-detected merchant via `services.suggest_keyword()`) and a `source` of `rule`/`ai`/`null`. Execute's `rules` param (list of `{keyword, category_id}`) upserts them. Resolution order in AI mode: matching mapped value → exact-signature `ai_categories` override → learned rule → Uncategorized. Manage via `GET/POST /api/wallets/import-rules/` and `DELETE /api/wallets/import-rules/{id}/` (or Django admin).

## Data Storage & Backups

The SQLite DB lives outside the repo at `~/.budgeting-app/db.sqlite3`, set via `DATABASE_URL` in `backend/.env` (default falls back to `backend/db.sqlite3` if unset). Two management commands, both write under `~/.budgeting-app/`:
- `python manage.py backup_db` — sqlite3 online-backup copy to `~/.budgeting-app/backups/`, prunes anything older than 30 days (keeps at least 5)
- `python manage.py export_all` — human-readable per-entity CSV dump to `~/.budgeting-app/exports/`

A launchd agent (`scripts/`, installed via `scripts/install-backup-agent.sh`) runs `backup_db` daily. Full self-host setup/restore steps: see `RUNBOOK.md`.

## Currencies

Supported: `usd`, `eur`, `gbp`, `pln`. A transaction's currency must match its wallet's currency (enforced in `TransactionSerializer.validate()`).

## Pending Features

Next up (see ROADMAP.md for full build order):
- AI auto-categorization, receipt scan, budget recommendations, chat
- Over-budget alerts
- Auth & account management
- Feature flags
- Production readiness

---
> Source: [PatrykJamroz/budgeting-app](https://github.com/PatrykJamroz/budgeting-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
