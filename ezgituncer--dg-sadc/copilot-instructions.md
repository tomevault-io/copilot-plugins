## dg-sadc

> This file is the project guide for Claude Code. It is read automatically at the start of every session. Keep it short — only what's needed to understand the project.

# CLAUDE.md

This file is the project guide for Claude Code. It is read automatically at the start of every session. Keep it short — only what's needed to understand the project.

## Project Overview

**Workload Tracking Application** — an internal web application that lets a corporate team log their daily workload by category and produces monthly/yearly reports.

**Target users:** A 20–50 person company (roles: Director, Engineering Manager, Tech Lead, Worker, HR, QA Specialist)

**Main flow:**
1. Each employee logs how many hours they spent on what each day (Activity Type → Category → Project + Task Type + hours)
2. Managers view monthly/yearly reports and can see who is below target relative to expected working days
3. Admins manage users, projects, categories, and activity types

## Tech Stack

- **Frontend:** Angular v20 (latest, signals + standalone components)
- **Backend:** Python FastAPI + SQLAlchemy 2.x + Alembic
- **Database:** PostgreSQL 15+
- **Auth:** JWT (access + refresh token, email + password)
- **Test:** pytest (backend), Jasmine + Karma (frontend)

## Domain Model

### Roles (fixed, code immutable)

| Code | Name | Permissions |
|---|---|---|
| `ADMIN` | Director / System Administrator | Full access; resets user passwords |
| `HR` | HR | User CRUD, lookups CRUD, sees reports (working days **read-only**) |
| `MANAGER` | Engineering/Product Manager | User CRUD, lookups CRUD, reports, working days **edit** |
| `TECH_LEAD` | Tech Lead | Same as Manager (including working days edit) |
| `QA_SPECIALIST` | QA Specialist | Same as Manager (including working days edit) |
| `WORKER` | Worker | Only enters their own workload; views listings read-only |

**Only ADMIN** can reset another user's password. There is NO forgot password / register flow.

### Hierarchy (mandatory rule)

```
Director (System Administrator)
  └── HEM (Head of Engineering)
        └── Engineering Manager (EM)
              └── Worker / Tech Lead
```

- **Every Worker must report to an Engineering Manager.**
- **Every Engineering Manager must report to a HEM.**
- HEM is a single person, reporting to the Director.
- Product Manager can report directly to the Director (not to HEM).
- HR and QA Specialist are **outside** the hierarchy tree — they report to the Director and are shown in their own panels.
- This rule is enforced via `users.manager_account_id`, not by position.

### Workload Entry structure

```
{
  account_id, work_date, hours_spent (decimal, e.g. 2.5),
  activity_type_id,    -- 1=Project, 2=Non-Project, 3=Self Improvement
  category_id,         -- from the category list belonging to the activity_type
  project_id,          -- ONLY filled when activity_type=1, NULL otherwise
  task_type_id,
  task_description (text),
  status (enum: ongoing | completed | blocked),
  complexity (enum: low | medium | high),
  quantity (int, optional)
}
```

**Important rules:**
- Activity Type is a CRUD-able table (default 3 records: PROJECT, NON_PROJECT, SELF_IMP). IDs 1, 2, 3 are stable.
- **There are 3 separate category tables:** `project_categories`, `non_project_categories`, `self_imp_categories`. The same name can exist across different lists (separate PKs).
- Category is paired with `activity_type` — `category_id` alone is not unique; the `(activity_type_id, category_id)` pair is what's meaningful.
- **Decimal hours:** 0.25 step (0.25, 0.5, 0.75, 1, 1.25 ...). NO daily upper limit.
- **30-day edit window:** An entry can only be edited / deleted if its `work_date` falls within the last 30 days.
- **Hard delete** for workload entries, **soft delete** (`is_active=false`) for users and lookups.

### Expected Working Days

A 12-month array per year (`expected_working_days[year][monthIndex]`). Admin/Manager/Tech Lead/QA edit it; HR can only view. Yearly report coloring is based on this value:

```
target = expected_working_days × 8
ratio  = user_total_hours / target

green   if ratio > 1.0
none    if ratio == 1.0
yellow  if ratio >= 0.5
red     if ratio < 0.5
```

If a project filter is selected, the same coloring logic applies (Non-Project and Self Imp are no longer separate projects — they were moved to activity type).

## Folder Structure

```
/backend
  /alembic               # migrations
  /app
    /api/v1              # FastAPI routers (auth, users, workload, reports, lookups)
    /core                # config, security (JWT), database
    /models              # SQLAlchemy models
    /schemas             # Pydantic schemas
    /services            # business logic
    /tests               # pytest
  /requirements.txt
  /alembic.ini

/frontend
  /src/app
    /core                # auth, http interceptor, guards
    /features
      /auth
      /dashboard
      /workload-entry
      /workload-list
      /yearly-report
      /users
      /lookups
    /shared              # common components (avatar, badge, etc.)
  /angular.json

/mockup
  workload-app.jsx       # Mock prototype (reference, do not touch)

/docs
  TASK.md                # Step-by-step to-do list
  CLAUDE.md              # This file
```

## Code Conventions

### Backend (Python/FastAPI)

- **Routes:** `/api/v1/<resource>` format. Plural noun. e.g. `/api/v1/workload-entries`.
- **snake_case** everywhere (model fields, response fields). The frontend converts to camelCase.
- **Pydantic v2.** `model_config = ConfigDict(from_attributes=True)`.
- **Do not use response wrappers** — return data directly. Use FastAPI's native HTTPException for errors.
- **Soft delete:** set `is_active=false`, never delete. Default queries hide inactive rows; expose them via `?include_inactive=true`.
- **Service layer:** business logic lives here. Routers handle request/response only, validation in Pydantic, DB work in services.
- **Async** everywhere (`async def`, `AsyncSession`).

### Frontend (Angular v20)

- **Standalone components** (no NgModules).
- **Signals** for state. RxJS observables only for HTTP.
- **OnPush** change detection on every component.
- **camelCase** fields (backend snake_case → frontend camelCase auto-conversion in the interceptor).
- **Smart/dumb component split:** feature folders own smart components, shared/ holds dumb components.
- **Routing:** lazy-load every feature.
- **Auth guard:** `canActivate` on every authenticated route.
- **HTTP interceptor:** add JWT, log out on 401.

### Database

- Table names are **snake_case plural** (users, workload_entries, project_categories).
- PK is `BIGSERIAL` (matches the mock). Foreign keys use the `_id` suffix.
- `created_at`, `updated_at` on every table (timestamp with tz, default now()).
- Tables with soft delete include `is_active BOOLEAN NOT NULL DEFAULT true`.
- Enum columns use **CHECK constraints**: `status` (`ongoing`/`completed`/`blocked`), `complexity` (`low`/`medium`/`high`).
- Foreign keys use **ON DELETE RESTRICT** (so workload entries and users can't be silently dropped).

## Mock Prototype Reference

`/mockup/workload-app.jsx` is the live reference for all UI and business rules. Run the relevant screen and observe the behavior before developing. Mock data and form flows live in this file.

**What will not change from the mock:**
- Dashboard (org tree view)
- Workload entry form flow (Activity type → Category → Project flow)
- Yearly Report matrix (color rules, expand/collapse breakdown)
- Listings (filter top → KPI → 3 charts → table)
- Users (split view + right-panel form)
- Management (6 tabs: projects, activity types, project categories, non-project categories, self-imp categories, task types)
- 30-day edit window
- Working days editor

**Things missing from the mock that will be added:**
- Password hashing (bcrypt)
- JWT token expiration & refresh
- Audit log (who changed what, when — optional)
- Email notifications (optional)
- File upload (optional — attachments on workload)

## Test Accounts (seeded for development)

```
admin@company.com           / admin123    (ADMIN, Director)
hr.manager@company.com      / hr123       (HR)
eng.manager@company.com     / mgr123      (MANAGER, Engineering Manager)
frontend.lead@company.com   / tl123       (TECH_LEAD)
qa.lead@company.com         / qa123       (QA_SPECIALIST)
developer1@company.com      / pass123     (WORKER)
```

These seed records are not created in production; they are only seeded when the env var `app.config.SEED_USERS=true` is set.

## Running the App

### Backend

```bash
cd backend
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # DB credentials, JWT secret
alembic upgrade head
uvicorn app.main:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm start  # Angular dev server, port 4200
```

## Important Notes

- **Turkish UI strings:** the UI is in Turkish (e.g. Workload entry → "Workload girişi", Yearly Report → "Yıllık Rapor"). Code and API are fully in English.
- **Stable IDs from the mock:** activity_type_id 1=PROJECT, 2=NON_PROJECT, 3=SELF_IMP. Insert with these IDs in the migration.
- **Hierarchy validation** must be enforced on the backend: a worker without a manager cannot be saved. The only exception is the Director (manager_account_id NULL).
- **CORS:** `http://localhost:4200` in dev, the Angular deploy URL in prod.
- **Timezone:** server TZ='UTC'; the frontend renders in the user's local time.

## First Steps in a New Session

When you start a new session: read **TASK.md** first to find out which task we're on, then move to the relevant code files.

---
> Source: [ezgituncer/dg-sadc](https://github.com/ezgituncer/dg-sadc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
