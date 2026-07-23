## vetra-companies-templates-db

> Do not modify Vetra Companies/Templates database without explicit user request


# Vetra Companies & Templates — Database Freeze (STRICT)

**User data in `vetra_companies` / `vetra_templates` is production-critical. Do not risk it.**

Unless the user **explicitly** asks to change the database layer, do **not** modify:

## Backend (Neon / Postgres) — FORBIDDEN

- Tables: `vetra_companies`, `vetra_templates`
- DB logic in `backend/app/db/neon.py` for company/template CRUD, migrations, bootstrap, or legacy copy
- API routes: `/api/neon/vetra/companies`, `/api/neon/vetra/templates` in `backend/app/api/neon_routes.py`
- Schemas: `VetraCompany*`, `VetraTemplate*` in `backend/app/models/project_schemas.py`
- SQL docs/migrations that define or alter these two tables
- One-off scripts that INSERT/UPDATE/DELETE vetra rows (read-only audit scripts are OK)

## Allowed without asking

- Frontend UI/UX for Companies, Templates, Outreach views
- Frontend workspace sync, cache, and API **clients** when fixing load/display bugs **without** changing API contract or DB shape
- Read-only diagnostics (e.g. `backend/scripts/audit_vetra_data.py`)
- Outreach features that **read** company/template data

## Required before any DB change

- User must clearly request a database/schema/API persistence change in the current message
- Keep `vetra_companies` (introduction only) and `vetra_templates` (subject/body only) as **separate** stores
- Never merge lists or share one table again
- Never auto-seed, auto-delete, or overwrite server data from empty client state

## Frontend safety rules

- Never apply an empty server list over cached/local snapshot that has records
- Persist successful fetches to scoped localStorage as backup
- Fix auth/sync races without touching backend persistence

---
> Source: [ZhangBoyanEvens/Vetra](https://github.com/ZhangBoyanEvens/Vetra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
