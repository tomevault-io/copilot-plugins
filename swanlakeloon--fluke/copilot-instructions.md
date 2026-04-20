## fluke

> - **Always save** walkthroughs, implementation plans, task lists, QA reports, and any other AI-generated documents to the `ai-artifacts/` folder in the project root

# CLAUDE.md — Fluke Project Rules

## AI Artifacts
- **Always save** walkthroughs, implementation plans, task lists, QA reports, and any other AI-generated documents to the `ai-artifacts/` folder in the project root
- Example: `ai-artifacts/walkthrough.md`, `ai-artifacts/implementation_plan.md`, `ai-artifacts/task.md`
- Do NOT save these to tmp, Desktop, or any other location

## Python
- **Always run Python with `uv`**: `uv run scripts/foo.py`
- Never use `python3` or `pip` directly
- Scripts use inline `# /// script` dependency declarations (PEP 723)

## Project Name
**Fluke** — Open-source ALPR data explorer (a play on Flock)

## Stack
- **Backend:** PocketBase v0.25 at `http://127.0.0.1:8090`
- **Frontend:** React + Vite at `http://localhost:5173`
- **Languages:** Python (scripts), JavaScript/JSX (frontend)
- **Package manager:** npm (frontend), uv (Python)

## Directory Structure
```
plate-database/
├── backend/          # PocketBase binary + pb_data/
├── frontend/         # React + Vite app
│   └── src/
│       ├── api/      # PocketBase client
│       ├── components/
│       ├── hooks/    # useAuth
│       ├── pages/    # Login, Search, admin/*
│       └── utils/    # colorMap, groupSightings
├── scripts/          # Python import/backup/setup scripts
├── backups/          # PocketBase snapshots (gitignored)
└── data/             # Source CSV files (gitignored)
```

## Common Commands
```bash
# Start PocketBase
./backend/pocketbase serve

# Start frontend dev server
cd frontend && npm run dev

# Import CSV
uv run scripts/import_csv.py ./data/file.csv

# Backup database
uv run scripts/backup.py

# Fix PocketBase API rules
uv run scripts/fix-api-rules.py
```

## PocketBase
- Superuser: `admin@local.dev` (local dev only)
- Auth collection: `users` with custom `role` select field (`user` | `admin`)
- API rule pattern: `@request.auth.role ?= "admin"` (use `?=` not `=` for select fields)
- **Do NOT sort by autodate fields** (`created`, `updated`) — PB v0.25 returns 400 when API rules are set

## Color Codes
Valid values for the `color` field: `BR, GR, BK, BL, TN, SL, R, WH, GN, GD, PU`
When adding a new color, update ALL of:
1. `scripts/import_engine.py` — `VALID_COLORS` set
2. `frontend/src/pages/admin/CsvUpload.jsx` — `VALID_COLORS` set
3. `frontend/src/utils/colorMap.js` — `COLOR_MAP` object
4. PocketBase schema via API script

## ICE Field Values
`Y` = ICE vehicle confirmed, `N` = not confirmed, `HS` = Homeland Security interest
`searchable` is auto-derived as `true` when `ice` is `Y` or `HS`

## Frontend Conventions
- Badge colors: `badge-warning` (yellow) for flagged/alert states, `badge-muted` for neutral
- Never use `badge-danger` — it is not defined in the design system
- Avoid `sort: '-created'` or `sort: '-updated'` in PocketBase queries (causes 400)
- Location filter uses fuzzy `~` operator (contains), not exact match

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SwanLakeLoon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
