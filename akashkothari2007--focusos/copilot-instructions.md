## focusos

> This file is the single source of truth for working on this codebase. Update it whenever architecture or conventions change.

# FocusOS — Claude Reference

This file is the single source of truth for working on this codebase. Update it whenever architecture or conventions change.

---

## Claude Session Management (MCP)

When starting work on a FocusOS task/subtask, **automatically**:
1. Use `mcp__focusos__start_session` for the relevant todo (or freeform if no matching todo)
2. When a subtask is completed, use `mcp__focusos__complete_subtask` to check it off
3. Append meaningful notes with `mcp__focusos__append_session_notes` summarizing what was done

Do this proactively — don't wait for the user to ask.

---

## Project Overview

Personal productivity + job-search dashboard. Backend is production on Railway, frontend is a local Vite dev server proxying to Railway (or localhost when developing backend).

---

## Tech Stack

| Layer | Details |
|-------|---------|
| Backend | FastAPI (Python 3.11), APScheduler for scheduled jobs |
| DB | PostgreSQL 16, psycopg3 (binary), `row_factory=dict_row` — rows are dicts |
| AI | OpenAI API via `backend/ai.py` → `chat_json(messages)` returns parsed JSON dict |
| Frontend | React 18 (Vite), plain CSS in `testfrontend/src/styles/app.css` |
| Dev env | Docker Compose (backend + db containers) |
| Deploy | Railway (backend auto-deploys from main) |

---

## Running

```bash
# Backend + DB
docker compose up -d --build

# Frontend dev
cd testfrontend && npm run dev   # → http://localhost:5173
```

Backend on port 8000. Vite proxies `/api` and `/auth` to backend (see `vite.config.js`).

---

## Environment Variables

**`backend/.env`:**
```
DATABASE_URL=postgresql://focusos:focusos@db:5432/focusos
FOCUSOS_API_KEY=<shared secret>
OPENAI_API_KEY=<key>
MS_CLIENT_ID=
MS_CLIENT_SECRET=
MS_REDIRECT_URI=
MS_USER_EMAIL=
```

**`testfrontend/.env`:**
```
VITE_FOCUSOS_API_KEY=<same shared secret>
```

---

## Auth / Middleware

Every request to the backend (except `/health`, `/db`, `/auth/login`, `/auth/callback`) must include:
```
X-API-Key: <FOCUSOS_API_KEY>
```
The `api_key_middleware` in `main.py` enforces this and returns 401 otherwise.

Frontend sends it via `api.js`:
```js
headers: { 'X-API-Key': import.meta.env.VITE_FOCUSOS_API_KEY }
```

---

## Backend File Map

```
backend/
├── main.py              # App factory, CORS, middleware, APScheduler startup
├── db.py                # get_conn() — psycopg3 context manager, dict_row
├── ai.py                # chat_json(messages) → dict  (OpenAI wrapper)
├── prompts.py           # summary_messages(), analysis_messages(), resume_messages()
├── scheduler.py         # run_email_scan() — called 8am + 6pm ET
├── requirements.txt     # fastapi, uvicorn[standard], psycopg[binary], python-dotenv, openai, httpx, apscheduler
├── Dockerfile
├── routers/
│   ├── todo_router.py
│   ├── session_router.py
│   ├── job_router.py       # Heavy: background AI tasks, resume generation pipeline
│   ├── doc_router.py
│   ├── profile_router.py
│   ├── habit_router.py
│   └── email_router.py     # Microsoft Graph OAuth + email scanning
├── models/
│   ├── todo_models.py      # CreateTodo, UpdateTodo, Link, ReorderTodos
│   ├── session_models.py   # EndSession, StartFreeformSession
│   ├── job_models.py       # CreateJob, UpdateJob, AnalyzeJob, GenerateResumeBody
│   ├── doc_models.py
│   ├── profile_models.py
│   └── habit_models.py     # CreateHabit, UpdateHabit, ToggleHabitLog
└── jobs/
    ├── latex_handler.py    # parse_latex(content) → {experiences, projects}
    └── resume_injector.py  # inject_changes(latex, ai_data, project_plan, experience_plan) → latex str
```

---

## Frontend File Map

```
testfrontend/src/
├── App.jsx              # Router, navbar, activeSession state, SessionBar mount
├── api.js               # All API calls — single source of truth for endpoints
├── styles/app.css       # All styles (single file, ~1800 lines)
├── pages/
│   ├── Todos.jsx        # 3-col layout: TodayStrip | todo list | HabitTracker
│   ├── Calendar.jsx     # Week calendar view (Google Calendar style)
│   ├── Metrics.jsx      # Session history + habit metrics
│   ├── Jobs.jsx         # Job portal, AI analysis UI
│   ├── Docs.jsx         # LaTeX resume editor
│   └── Profile.jsx      # Projects, experiences, skills editor
└── components/
    ├── TodoCard.jsx      # Expandable card: subtasks, links, description, session start
    ├── AddTodoForm.jsx   # Inline add form at top of list
    ├── SessionBar.jsx    # Fixed bottom bar when session is active
    ├── HabitTracker.jsx  # 7-day habit grid with streaks (sidebar)
    └── TodayStrip.jsx    # Day timeline strip (left sidebar on Todos page)
```

---

## Active Session State

Managed in `App.jsx`:
```js
const [activeSession, setActiveSession] = useState(null);
// Shape: { sessionId, todoId, todoTitle, startedAt }
```
Restored on load via `api.getActiveSession()`. Passed down to `Todos`, `Calendar`, `SessionBar`.

Only one session can be open at a time globally — backend enforces this with a 409 if you try to start another.

---

## Database Schema

### `todos`
| Column | Type | Notes |
|--------|------|-------|
| id | bigint PK | auto-increment |
| title | text | required |
| status | text | `pending` \| `done` |
| description | text | nullable |
| subtasks | jsonb | `[{id, title, status, order}]` |
| due_date | timestamptz | nullable |
| sort_order | int | used for drag reorder of undated todos |
| links | jsonb | `[{title, url}]` |
| created_at | timestamptz | default now() |

Order: due dated first (ASC), then undated by `sort_order ASC`.

### `sessions`
| Column | Type | Notes |
|--------|------|-------|
| id | bigint PK | |
| todo_id | bigint | nullable — null for freeform sessions |
| title | text | freeform name, or todo title (coalesced on read) |
| started_at | timestamptz | default now() |
| ended_at | timestamptz | null = in-progress |
| seconds_spent | int | auto-calculated on end |
| notes | text | |

FK: `todo_id → todos.id` (cascade delete)

### `jobs`
| Column | Type | Notes |
|--------|------|-------|
| id | bigint PK | |
| title | text | required |
| company | text | required |
| status | text | `saved` \| `applied` \| `interview` \| `rejected` |
| link | text | job posting URL |
| description | text | raw job description |
| summary | text | AI-generated on create (background task) |
| keywords | jsonb | AI-extracted keywords (`string[]`) |
| analysis_status | text | `idle` \| `summarizing` \| `analyzing` \| `generating_resume` \| `done` \| `error` |
| created_at | timestamptz | |

### `job_analysis`
| Column | Type | Notes |
|--------|------|-------|
| job_id | bigint PK | 1:1 with jobs |
| input_doc_id | bigint | resume used (→ docs) |
| output_doc_id | bigint | tailored resume (→ docs) |
| match_score | int | 0–100 |
| keywords | jsonb | `string[]` |
| suggestions | jsonb | structured AI suggestions (experiences + projects) |
| updated_at | timestamptz | |

FK: `job_id → jobs.id` (cascade), `input/output_doc_id → docs.id` (set null)

### `docs`
| Column | Type | Notes |
|--------|------|-------|
| id | bigint PK | |
| title | text | required |
| content | text | LaTeX source |
| is_primary | boolean | marks the base resume used for analysis |
| created_at | timestamptz | |

### `profile`
| Column | Type | Notes |
|--------|------|-------|
| id | int | always 1 (single row) |
| projects | jsonb | `[{title, description, link, tech[]}]` |
| experiences | jsonb | `[{company, role, date, location, bullets[]}]` |
| skills | text | freeform skills string |

### `habits`
| Column | Type | Notes |
|--------|------|-------|
| id | int PK | |
| name | varchar(100) | |
| frequency | int | 7 = daily, lower = weekly target |
| is_active | boolean | default true |
| created_at | timestamptz | |

### `habit_logs`
| Column | Type | Notes |
|--------|------|-------|
| id | int PK | |
| habit_id | int | FK → habits (cascade) |
| log_date | date | |
| completed | boolean | toggled via upsert |

Unique constraint on `(habit_id, log_date)`.

### `email_accounts`
| Column | Type | Notes |
|--------|------|-------|
| id | int PK | |
| email | varchar(255) | |
| access_token | text | Microsoft Graph OAuth |
| refresh_token | text | |
| expires_at | timestamptz | |
| created_at | timestamptz | |

### `scanned_email_ids`
| Column | Type | Notes |
|--------|------|-------|
| email_id | text PK | Microsoft Graph message ID |
| scanned_at | timestamptz | |

Created at startup in `main.py` if not exists.

---

## API Routes

All routes: prefix `/api/v1`. Auth via `X-API-Key` header.

### Todos
```
GET    /todos                     ?status=pending|done
POST   /todos                     {title, description?, due_date?, subtasks?}
PATCH  /todos/{id}                partial update
DELETE /todos/{id}                204
POST   /todos/reorder             {ids: int[]}
GET    /todos/{id}/sessions
POST   /todos/{id}/sessions/start  → 409 if session already open
```

### Sessions
```
GET    /sessions/active            null if none open
POST   /sessions/start             {title, notes?}  freeform (no todo)
PATCH  /sessions/{id}/end          {notes?}  auto-calculates seconds_spent
PATCH  /sessions/{id}/notes        {notes}  update notes on any session (incl. ended)
DELETE /sessions/{id}              204
GET    /sessions/today             ?start=ISO&end=ISO  (UTC bounds)
GET    /sessions/week              ?start=ISO&end=ISO  (UTC bounds)
POST   /todos/quick-session        {project}  Siri shortcut — fuzzy match + start (200 skipped if active)
POST   /sessions/quick-end         Siri shortcut — end active session (200 skipped if none)
```

### Jobs
```
GET    /jobs                       ?status=saved|applied|interview|rejected
GET    /jobs/{id}                  includes joined job_analysis row
POST   /jobs                       {title, company, status?, link?, description?}
                                   → triggers AI summary in background if description given
PATCH  /jobs/{id}
DELETE /jobs/{id}                  204
POST   /jobs/{id}/analyze          {input_doc_id}  → background AI analysis
DELETE /jobs/{id}/analysis         204
POST   /jobs/{id}/generate-resume  {selected_experiences?, selected_projects?}  → background
```

Jobs `analysis_status` flow:
`idle` → `summarizing` → `idle` (summary done) → `analyzing` → `done` → `generating_resume` → `done`

### Docs
```
GET    /docs
POST   /docs                       {title, content?}
PATCH  /docs/{id}                  {title?, content?}
PATCH  /docs/{id}/set-primary      marks as primary, clears others
DELETE /docs/{id}                  204
```

### Profile
```
GET    /profile
PATCH  /profile                    {projects?, experiences?, skills?}
```

### Habits
```
GET    /habits                     ?active=true|false
POST   /habits                     {name, frequency}
PATCH  /habits/{id}                {name?, is_active?, frequency?}
DELETE /habits/{id}                204
GET    /habits/logs                ?days=7&today=YYYY-MM-DD  → grid + streaks
POST   /habits/logs/toggle         {habit_id, log_date}
```

### Auth / Email
```
GET    /auth/login                 redirect to Microsoft OAuth
GET    /auth/callback              OAuth callback
GET    /auth/status                {connected: bool, email?}
POST   /auth/disconnect
```

### Health
```
GET    /health    → {"ok": true}
GET    /db        → {"db": "connected"}
```

---

## AI Pipeline (Jobs)

1. **Create job** (with description) → `generate_summary_and_keywords` runs in background
   - Calls `chat_json(summary_messages(description))` → `{summary, keywords}`
   - Stores on job row, sets `analysis_status = 'idle'`

2. **Analyze job** → `run_analysis(job_id, input_doc_id)` runs in background
   - Fetches job summary/keywords + resume content + profile
   - Parses LaTeX with `parse_latex()` → structured experiences/projects
   - Calls `chat_json(analysis_messages(...))` → `{match_score, experiences, projects}`
   - Stores in `job_analysis`, sets `analysis_status = 'done'`

3. **Generate resume** → `generate_resume_bg(job_id, overrides)` runs in background
   - Reads analysis suggestions, builds `experience_plan` / `project_plan`
   - Calls `chat_json(resume_messages(...))` → rewritten bullets
   - Calls `inject_changes(latex, ai_data, project_plan, experience_plan)` → final LaTeX
   - Saves new doc to `docs`, links as `output_doc_id` in `job_analysis`

`chat_json` is in `backend/ai.py`. All prompts are in `backend/prompts.py`.

---

## Frontend Patterns

### API calls
All calls go through `testfrontend/src/api.js`. Add new endpoints there. Vite proxies `/api` and `/auth` to the backend (Railway in dev, localhost if running docker compose).

### CSS
Single file: `testfrontend/src/styles/app.css`. Use existing classes — don't add inline styles.

**Color palette:**
- Primary: `#6366f1` (indigo) / `#7c3aed` (violet)
- Secondary: `#14b8a6` (teal) — used for links
- Danger: `#e11d48` (rose)
- Success: `#16a34a` / `#22c55e` (green)
- Warning: `#f59e0b` (amber)
- Background: `#eef0f8`
- Text: `#1a1a2e` (primary) / `#475569` (secondary) / `#94a3b8` (muted)

**Reusable utility classes:**
- `.glass-card` — frosted glass card
- `.btn`, `.btn-primary`, `.btn-sm`, `.btn-done`, `.btn-end`, `.btn-tag`
- `.input`, `.input-sm`, `.textarea`, `.select`
- `.page`, `.page-title` — legacy centered layout (Jobs/Docs/Profile)

**Page layouts:**
- Todos: 3-col grid `280px 1fr 280px` (TodayStrip | main | HabitTracker)
- Calendar: full-height flex, sticky header row, scrollable time grid
- Metrics: 2-col grid `1fr 1fr`

### Session state
Passed as props `activeSession` / `setActiveSession` from `App.jsx` down to `Todos`, `Calendar`, `SessionBar`. Always check `activeSession` before offering to start a new session.

---

## Conventions

- Backend: all DB rows returned as dicts (psycopg3 `dict_row`). Use `model_dump(exclude_none=True)` for PATCH routes.
- JSONB fields (`subtasks`, `links`, `suggestions`, etc.) must be `json.dumps(...)` before inserting.
- Background tasks use FastAPI `BackgroundTasks` (in-process, no Redis). Heavy jobs log with `log = logging.getLogger("router_name")`.
- Frontend date handling: always convert local dates to UTC ISO strings before sending to the session endpoints (`getTodaySessions`, `getWeekSessions`).
- Habit `frequency` field: 7 = treat as daily, anything less = weekly target (tracked by `week_count`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akashkothari2007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akashkothari2007)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
