## project-reference

> >-


# ArmillaryLab — Project Reference Cache

> **Last updated:** 2026-05-22
> **App version:** 2.2.0 (`APP_VERSION` in `app.py`)
> **Purpose:** Save agent discovery time and token cost. Read this FIRST, explore SECOND.
> **Maintenance:** MANDATORY — before committing, if you modified app.py (models/routes/helpers/database schema), any template, astro_utils.py, **conditions_utils.py**, **calibration_utils.py**, time_utils.py, nina_integration.py, config/*.py, requirements.txt, or .env.example in a way that changes the project's structure or API surface (new/removed/renamed models, columns, routes, templates, env vars, dependencies), you MUST update the relevant section(s) below and bump the "Last updated" date. A stop-hook also enforces this as a safety net.

---

## 1. Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Backend | Flask 3.x | Single-module app (`app.py`) |
| ORM | SQLAlchemy 2.x + Flask-SQLAlchemy 3.x | All models in `app.py` |
| Database | SQLite (dev) / PostgreSQL (prod) | Configured via `DATABASE_URL` env var |
| Migrations | Custom `config/migration.py` | Full bidirectional SQLite↔PostgreSQL migrator (not Alembic) |
| Astronomy | astropy 6.x + astroplan 0.10+ | Window/altitude + moon phase (Night Conditions) |
| External APIs | Open-Meteo, 7Timer | Weather & seeing (no API keys; server-side fetch in `conditions_utils.py`) |
| Templates | Jinja2 (Flask built-in) | Dark-themed Bootstrap 5 |
| Charts | Chart.js (vendored UMD) | Altitude-over-night plot |
| Frontend | Bootstrap 5 + Bootstrap Icons | No custom CSS/JS files — all inline |
| Server | gunicorn (prod), Flask dev server (dev) | Port 5000 |
| Container | Docker + docker-compose | Single `web` service |

---

## 2. Directory Structure

```
astroplanner/
├── app.py                    # Flask app: ALL models + ALL routes + helpers
├── astro_utils.py            # compute_target_window() — core astronomy
├── conditions_utils.py       # Night conditions: moon phase, weather, seeing, channel suggestion
├── calibration_utils.py      # Calibration frame tracking: suggestions, status, export totals
├── time_utils.py             # utc_to_local() + template filter registration
├── nina_integration.py       # NINA XML sequence + filter profile parsing
├── cli.py                    # Dev server entry: python cli.py
├── config/
│   ├── database.py           # configure_app() — DB URI + engine options
│   └── migration.py          # DatabaseMigrator class — bidirectional SQLite↔PostgreSQL
├── templates/                # 17+ Jinja2 templates (see §6)
├── static/
│   ├── css/                  # bootstrap.min.css, bootstrap-icons.min.css
│   ├── js/                   # bootstrap.bundle.min.js, chart.umd.js
│   ├── fonts/                # bootstrap-icons.woff, .woff2
│   └── images/               # armillarylab-logo.png, favicon.ico
├── tests/
│   ├── __init__.py
│   └── test_app.py           # pytest suite (~15 tests)
├── uploads/                  # User-uploaded final images (gitignored)
├── cache/                    # Weather/seeing forecast cache (gitignored)
│   └── conditions/           # JSON cache files keyed by lat/lon
├── requirements.txt          # 11 dependencies
├── .env / .env.example       # Observer config, DB URL, secrets
├── Dockerfile                # python:3.12-slim, gunicorn
├── docker-compose.yml        # Single web service, volume mounts
├── run_tests.py              # pytest runner script
├── armillarylab.db           # Bundled SQLite showcase data (tracked; omit via delete + flask init-db for blank)
└── README.md
```

---

## 3. Database Models (all in `app.py`)

> Models define the database schema. SQLAlchemy columns map directly to DB columns.
> Schema changes happen by modifying model classes, then `config/migration.py` handles syncing.

### GlobalConfig (`global_config`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| observer_lat | Float, default 24.7136 | Observer latitude |
| observer_lon | Float, default 46.6753 | Observer longitude |
| observer_elev_m | Float, default 600 | Elevation in meters |
| default_packup_time | String(5), default "01:00" | Default pack-up HH:MM |
| default_min_altitude | Float, default 30.0 | Default min altitude degrees |
| default_calibration_darks | Integer, default 0 | Default dark count per target |
| default_calibration_flats_per_channel | Integer, default 0 | Flats per plan channel |
| default_calibration_dark_flats_per_channel | Integer, default 0 | Dark flats per channel |
| default_calibration_bias | Integer, default 0 | Optional bias count |
| default_calibration_two_point | Boolean, default True | Split flats at midpoint + end |
| max_cloud_cover_pct | Integer, default 25 | Cloud cover % threshold for go/skip advice |
| timezone_name | String(64), default "Asia/Riyadh" | IANA timezone |
| updated_at | DateTime, default utcnow | |

Singleton row — `get_global_config()` creates one if missing. Replaces .env for observer settings in DB-backed config.

### TargetType (`target_types`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| name | String(64), unique, not null | "emission", "galaxy", "planetary", etc. |
| recommended_palette | String(16), not null | "SHO", "LRGB", etc. |
| description | Text | Why this palette works for this type |
| created_at | DateTime, default utcnow | |

Relationships: `object_mappings` → ObjectMapping (cascade all, delete-orphan)

### ObjectMapping (`object_mappings`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| object_name | String(128), unique, not null | "NGC 6960", "M31", etc. |
| target_type_id | Integer FK → target_types.id, not null | |
| created_at | DateTime, default utcnow | |

Relationships: `target_type` → TargetType. Used by `detect_target_type()` to auto-classify catalog objects.

### Filter (`filters`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| name | String(32), unique, not null | Short code: "H", "O", "S", "L", "R", "G", "B" |
| display_name | String(64) | "Hydrogen Alpha", etc. |
| astrobin_filter_id | Integer | AstroBin API filter ID |

Relationships: `palette_entries` → PaletteFilter, `wheel_slots` → FilterWheelSlot

### Palette (`palettes`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| name | String(32), unique, not null | "SHO", "LRGB", etc. |
| display_name | String(64) | |
| is_active | Boolean, default True | |

Relationships: `filters` → PaletteFilter (cascade all, delete-orphan), `targets` → Target

### PaletteFilter (`palette_filters`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| palette_id | Integer FK → palettes.id | |
| filter_id | Integer FK → filters.id | |
| weight | Float, default 1.0 | Relative weight for time allocation |
| default_sub_exposure_seconds | Float, default 300 | |

### FilterWheel (`filter_wheels`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| name | String(64), not null | |
| slot_count | Integer, not null, default 7 | |
| is_active | Boolean, default True | |

Relationships: `slots` → FilterWheelSlot (cascade all, delete-orphan)

### FilterWheelSlot (`filter_wheel_slots`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| wheel_id | Integer FK → filter_wheels.id | |
| position | Integer, not null | 1-based slot number |
| filter_id | Integer FK → filters.id (nullable) | |
| nina_filter_name | String(64) | NINA-specific name |
| physical_filter_brand | String(128) | |
| notes | Text | |

### Target (`targets`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| name | String(128), not null | |
| catalog_id | String(64) | e.g. "NGC 7000" |
| target_type | String(64) | "Nebula", "Galaxy", etc. (legacy text) |
| target_type_id | Integer FK → target_types.id | New FK reference to TargetType |
| ra_hours | Float, not null | Right ascension 0-24 |
| dec_deg | Float, not null | Declination -90 to +90 |
| notes | Text | Editable inline on detail page |
| pixinsight_workflow | Text | Editable inline on detail page |
| preferred_palette | String(64), default "SHO" | |
| palette_id | Integer FK → palettes.id | |
| packup_time_local | String(5), default "01:00" | HH:MM format |
| override_packup_time | String(5) | Per-target override |
| override_min_altitude | Float | Per-target override |
| calibration_tracking_enabled | Boolean, default False | Opt-in calibration tracking |
| override_calibration_darks | Integer, nullable | NULL → global |
| override_calibration_flats_per_channel | Integer, nullable | NULL → global |
| override_calibration_dark_flats_per_channel | Integer, nullable | NULL → global |
| override_calibration_bias | Integer, nullable | NULL → global |
| override_calibration_two_point | Boolean, nullable | NULL → global |
| final_image_filename | String(256) | |
| created_at | DateTime, default utcnow | |
| is_archived | Boolean, default False | |
| archived_at | DateTime | |
| completion_notes | Text | |

Relationships: `plans` → TargetPlan, `sessions` → ImagingSession, `calibration_captures` → CalibrationCapture, `calibration_checkpoint_skips` → CalibrationCheckpointSkip, `palette` → Palette

### CalibrationCapture (`calibration_captures`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| target_id | FK → targets | cascade delete |
| date | Date | Capture night |
| frame_type | String(16) | dark, flat, dark_flat, bias |
| channel | String(16), nullable | Required for flat/dark_flat |
| checkpoint | String(16), nullable | midpoint, end, manual |
| frame_count | Integer | |
| notes | Text | |

### CalibrationCheckpointSkip (`calibration_checkpoint_skips`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| target_id | FK → targets | |
| channel | String(16) | Plan channel name |
| frame_type | String(16) | flat or dark_flat |
| checkpoint | String(16) | midpoint or end |
| skipped_at | DateTime | Unique on (target_id, channel, frame_type, checkpoint) |

### TargetPlan (`target_plans`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| target_id | Integer FK → targets.id | |
| plan_json | Text, not null | JSON: channels, minutes, sub-exposure, weights |
| created_at | DateTime, default utcnow | |

### ImagingSession (`imaging_sessions`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| target_id | Integer FK → targets.id | |
| date | Date, not null | |
| channel | String(32), not null | Filter channel used |
| sub_exposure_seconds | Float, not null | |
| sub_count | Integer, not null | |
| notes | Text | |

---

## 4. Routes (all in `app.py`)

### Dashboard & Browsing
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/` | GET | `index()` | Home — active target cards with tonight's recommendation, window info |
| `/archived` | GET | `archived_targets()` | Lists completed/archived targets |
| `/imaging-logs` | GET | `imaging_logs()` | Cross-target session log grouped by date |

### Target CRUD
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/new` | GET, POST | `new_target()` | Create target + initial plan from palette |
| `/target/<id>` | GET | `target_detail(id)` | Main detail page — plan, window, chart, sessions, notes |
| `/target/<id>/edit` | GET, POST | `edit_target(id)` | Full target edit form |
| `/target/<id>/delete` | POST | `delete_target(id)` | Delete target + cascade |
| `/target/<id>/settings` | GET, POST | `target_settings(id)` | Per-target overrides (packup, min altitude, calibration) |
| `/target/<id>/archive` | POST | `archive_target(id)` | Mark complete |
| `/target/<id>/unarchive` | POST | `unarchive_target(id)` | Restore to active |
| `/target/<id>/clone` | POST | `clone_target(id)` | Deep-clone target + latest plan |

### Target AJAX APIs
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/<id>/update-notes` | POST | `update_target_notes(id)` | JSON: `{notes}` → save, return `{ok, notes}` |
| `/target/<id>/update-pixinsight-workflow` | POST | `update_target_pixinsight_workflow(id)` | JSON: `{pixinsight_workflow}` → save |

### Plan Management
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/<id>/plan/new` | POST | `new_plan(id)` | Create plan from chosen palette |
| `/target/<id>/plan/update` | POST | `update_plan(id)` | Save plan: per-channel minutes/subexp, custom filters, removals |

### Imaging Sessions
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/<id>/progress/add` | POST | `add_progress(id)` | Add ImagingSession |
| `/session/<id>/edit` | GET, POST | `edit_session(id)` | Edit session form |
| `/session/<id>/delete` | POST | `delete_session(id)` | Delete session |

### Calibration Frames
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/<id>/calibration/log` | POST | `log_calibration_capture(id)` | Log calibration capture |
| `/target/<id>/calibration/skip` | POST | `skip_calibration_checkpoint(id)` | Skip midpoint/end suggestion |
| `/calibration/skip/<skip_id>/restore` | POST | `restore_calibration_checkpoint(skip_id)` | Undo skip; restore suggestion banner |
| `/calibration/<id>/edit` | GET, POST | `edit_calibration_capture(id)` | Edit capture |
| `/calibration/<id>/delete` | POST | `delete_calibration_capture(id)` | Delete capture |
| `/api/target/<id>/calibration` | GET | `api_target_calibration(id)` | JSON status + suggestions |

### Export
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/<id>/export/nina` | POST | `export_nina_sequence(id)` | Download NINA XML sequence |
| `/target/<id>/export/astrobin-csv` | POST | `export_astrobin_csv(id)` | Download AstroBin CSV |

### Image Upload
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/target/<id>/upload-final` | POST | `upload_final_image(id)` | Upload/replace final image |
| `/uploads/<filename>` | GET | `uploaded_file(filename)` | Serve uploaded images |

### Filter/Palette/Wheel CRUD
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/filters` | GET | `filter_list()` | List all filters |
| `/filters/apply-preset-ids` | POST | `apply_preset_astrobin_ids()` | Bulk-apply AstroBin IDs from preset |
| `/filter/new` | GET, POST | `new_filter()` | Create filter |
| `/filter/<id>/edit` | GET, POST | `edit_filter(id)` | Edit filter |
| `/filter/<id>/delete` | POST | `delete_filter(id)` | Delete filter |
| `/palettes` | GET | `palette_list()` | List all palettes |
| `/palette/new` | GET, POST | `new_palette()` | Create palette with filter assignments |
| `/palette/<id>/edit` | GET, POST | `edit_palette(id)` | Edit palette |
| `/palette/<id>/delete` | POST | `delete_palette(id)` | Delete palette |
| `/filter-wheels` | GET | `filter_wheel_list()` | List filter wheels |
| `/filter-wheel/new` | GET, POST | `new_filter_wheel()` | Create wheel with N slots |
| `/filter-wheel/<id>/edit` | GET, POST | `edit_filter_wheel(id)` | Edit wheel + slot assignments |
| `/filter-wheel/<id>/delete` | POST | `delete_filter_wheel(id)` | Delete wheel |
| `/filter-wheel/<id>/activate` | POST | `activate_filter_wheel(id)` | Activate wheel (deactivates others) |

### Object Mapping Management
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/manage-object-mappings` | GET, POST | `manage_object_mappings()` | Map catalog names to target types |

### Settings & Presets
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/settings` | GET, POST | `settings()` | Global config via `GlobalConfig` model |
| `/settings/export-preset` | POST | `export_preset_web()` | Export filter/palette config as preset file |
| `/settings/import-preset` | POST | `import_preset_web()` | Import preset file (merge or replace) |
| `/settings/download-preset/<name>` | GET | `download_preset(name)` | Download a built-in preset file |

### JSON APIs
| URL | Methods | Function | Purpose |
|-----|---------|----------|---------|
| `/api/filters` | GET | `api_filters()` | JSON list of all filters |
| `/api/resolve` | GET | `api_resolve()` | Resolve catalog name → RA/Dec + target type |
| `/api/palette-recommendation` | GET | `api_palette_recommendation()` | Recommend palette for a target type |
| `/api/target/<id>/window` | GET | `api_target_window(id)` | JSON window info for target |
| `/api/conditions/<id>` | GET | `api_conditions(id)` | Moon phase, weather, seeing, wind/gust + cloud go-skip advice, combined session advice, channel suggestion (3-tier fallback) |
| `/api/target/<id>/calibration` | GET | `api_target_calibration(id)` | Calibration status + suggestions JSON |
| `/api/active-wheel` | GET | `api_active_wheel()` | JSON active filter wheel + slots |
| `/health` | GET | `health_check()` | Returns `{"status": "healthy"}` |

### Error Handlers
- **404**: Renders `404.html` or JSON
- **500**: Renders `500.html` or JSON

---

## 5. Utility Modules

### `astro_utils.py` — `compute_target_window()`
Core function. Takes RA/Dec + observer params, returns dict with:
- Sunset, dark start/end, meridian transit (local + UTC strings)
- Effective imaging window start/end after altitude constraint
- `altitude_profile`: list of `{time_label, alt_deg}` at 5-min intervals (sunset→dawn)
- `moon_altitude_profile`: same for moon
- `moon_rise_local`, `moon_set_local`
- `total_minutes`, `min_altitude_deg`, `midpoint_altitude_deg`
- Uses astropy Observer, EarthLocation, pytz for timezone conversion

### `time_utils.py`
- `utc_to_local(utc_dt, tz_name)` — converts UTC datetime to local timezone
- `register_time_filters(app)` — registers Jinja2 template filters including `format_hms_from_minutes`

### `conditions_utils.py` — Night Conditions
Moon phase, weather, seeing, and channel suggestion with caching + offline fallback:
- `compute_moon_info(tz_name)` → dict with phase_name, illumination_pct, emoji, next_full_moon, days_to_full (fully offline via astroplan)
- `fetch_openmeteo(lat, lon)` → 5-day hourly forecast from Open-Meteo (temp, humidity, clouds, wind, wind gusts)
- `fetch_7timer(lat, lon)` → 72h astronomical forecast from 7Timer (seeing, transparency)
- `suggest_tonight_channel(plan_data, moon_illum_pct, progress_by_channel)` → weighted scoring: `score = moon_weight * remaining_ratio`; Ha=1.0 (strongest NB), SII=0.7, OIII=0.3 (weakest NB), broadband=0.2 at full moon; all 1.0 at new moon
- `compute_wind_session_advice(weather, window_weather)` → go/caution/marginal/skip verdict from peak gusts (km/h), gust factor vs sustained wind, optional notes (peak gust hour in window)
- `compute_cloud_session_advice(weather, window_weather, max_cloud_pct)` → go/caution/marginal/skip verdict from peak cloud cover vs user's configured threshold
- `compute_session_advice(wind_session, cloud_session)` → combined overall verdict (worst of wind + cloud)
- Imaging-window weather aggregate includes `gust_factor`, `peak_gust_local`, and `gust_hour_stats` (counts of forecast hours with gust ≥35/45 km/h, gust ≥1.5× that hour’s wind, plus short hints)
- `get_tonight_conditions(...)` → orchestrator with 3-tier fallback (online → cache → offline moon → error); JSON includes `wind_session`, `cloud_session`, `session_advice`
- Cache: JSON files in `cache/conditions/`, keyed by lat/lon, max age 6h for weather

### `calibration_utils.py` — Calibration Frames (v2.2.0)
- `channel_light_frames(channel, progress_seconds)` → planned/done frames + ratio
- `get_calibration_status(config, plan_data, progress_seconds, captures, skips)` → summary dict
- `get_calibration_suggestions(...)` → midpoint/end flat/dark_flat action items
- `get_calibration_payload(...)` → `{summary, suggestions}` for UI/API
- `aggregate_calibration_for_export(captures)` → AstroBin prefill totals
- `format_suggestion_flash(suggestions)` → flash message after light logging
- `channel_calibration_badges(...)` → plan table Cal column labels

### `nina_integration.py`
- `generate_nina_sequence(target, plan_data, remaining_by_channel)` → NINA XML string
- `get_nina_filters_from_profile(profile_path)` → list of `{name, position}`

### `config/database.py` — `configure_app(app)`
Sets `SQLALCHEMY_DATABASE_URI` from env `DATABASE_URL`, configures engine options:
- **PostgreSQL**: `pool_pre_ping=True`, `pool_size=5`, `max_overflow=10`
- **SQLite**: absolute path, `NullPool`, WAL disabled by default on OneDrive paths, `journal_mode=DELETE` + `synchronous=FULL`, PRAGMA on connect
- **`sqlite_file_path()`** — resolved Path to the `.db` file

### `config/sqlite_health.py` — SQLite health (no auto-restore)
- `sqlite_db_info(path)` / `sqlite_has_core_schema(path)` — validate file has `targets` table
- `check_sqlite_database(path)` — remove WAL sidecars, validate; **never replaces the file**
- `find_best_sqlite_backup(project_dir)` — pick richest `.backup_*` (sessions + calibration)
- `restore_sqlite_from_backup(path, project_dir)` — **manual only** via `scripts/restore_db.py`
- Used at app startup (check only), `flask migrate-db` (check only), and `scripts/restore_db.py`

### `config/migration.py` — `DatabaseMigrator` class
Full bidirectional database migration engine for SQLite ↔ PostgreSQL:
- `DatabaseMigrator(source_config, target_config)` — context manager
- `migrate_database(validate_before, validate_after, backup_target)` → result dict
- `export_database(engine)` → dict of table_name → list of row dicts
- `import_database(engine, data)` → import result with counts
- `prepare_target_schema()` — validates target has all required tables
- `validate_database(engine)` → (bool, errors) — checks connectivity + required tables
- `validate_record_counts(exported_data)` → per-table source vs target counts
- `create_backup(db_config)` → backup path (SQLite file copy; PostgreSQL not implemented)
- Table dependency ordering: independent tables first (filters, palettes, etc.), then FKs (targets), then dependents (plans, sessions)
- Convenience function: `migrate_database(source_config, target_config, **kwargs)`

---

## 6. Templates

| Template | Extends | Key Purpose | Key Variables |
|----------|---------|-------------|---------------|
| `base.html` | — | Root layout, navbar, flash messages, Night Conditions popup (`/api/conditions`) with Overview (go/skip wind above channel suggestion) + Wind detail tab, dark theme | `app_name`, `app_version` |
| `index.html` | base | Target dashboard with tonight's recommendation | `tonight_pick`, `target_summaries`, `archived_summaries` |
| `target_detail.html` | base | Main target page (plan, chart, sessions, calibration, notes) | `target`, `plan_data`, `window_info`, `progress_*`, `calibration_*`, `palettes`, `astrobin_filter_map` |
| `target_form.html` | base | Create/edit target form | `target`, `global_config`, `palettes` |
| `target_settings.html` | base | Per-target overrides + calibration | `target`, `global_config`, `effective_calibration` |
| `edit_calibration.html` | base | Edit calibration capture | `capture`, `target`, `channels` |
| `edit_session.html` | base | Edit imaging session | `session`, `target`, `channels` |
| `imaging_logs.html` | base | Cross-target session logs | `stats`, `grouped_sessions` |
| `archived_targets.html` | base | Completed targets list | `targets` (or `archived_summaries`) |
| `settings.html` | base | Global config + calibration defaults + preset import/export | `config`, `presets` |
| `filters.html` / `filter_list.html` | base | Filter list + AstroBin preset modal | `filters`, `presets` |
| `filter_form.html` | base | Create/edit filter | `filter` |
| `palettes.html` / `palette_list.html` | base | Palette list with filter details | `palettes` |
| `palette_form.html` | base | Create/edit palette (dynamic channel rows) | `palette`, `filters` |
| `filter_wheels.html` / `filter_wheel_list.html` | base | Filter wheel list | `wheels` |
| `filter_wheel_form.html` | base | Create/edit wheel + slots | `wheel`, `filters` |
| `filter_wheel_activate_confirm.html` | base | Activation warning with affected targets | `wheel`, `affected_targets` |
| `manage_object_mappings.html` | base | Map catalog objects to target types | `target_types`, `mappings` |
| `404.html` / `500.html` | base | Error pages | — |

### `target_detail.html` Inline JavaScript (largest JS block)
- Plan calculations: `recalcFrames()`, frame/time/status badges
- H:M:S conversion: `parseHMS()`, `minutesToHMS()`, `formatHMS()`
- Altitude chart: Chart.js with 5 custom plugins (moon bar, window shading, current time line, meridian line, moon rise/set lines)
- Inline notes/workflow editing: `toggleNotesEdit()`, `saveNotes()`, `togglePixiEdit()`, `savePixiWorkflow()`
- Custom filter addition + auto-calculation
- Filter removal handling
- Calibration: action-item banners, log form prefill, channel toggle JS
- Auto-refresh current time marker every 60s

### `target_form.html` Inline JavaScript
- Catalog name resolver: `GET /api/resolve?name=...` for RA/Dec + target type
- Palette recommendation from target type map

### `palette_form.html` Inline JavaScript
- Dynamic channel rows: `addChannel()`, `removeChannel()`, `updateChannelCount()`

---

## 7. Environment Variables (legacy .env + GlobalConfig DB)

Settings are stored in the `GlobalConfig` DB model (single row). Legacy .env may still provide fallback values.

| Variable / Column | Default | Purpose |
|-------------------|---------|---------|
| `observer_lat` / `OBSERVER_LAT` | 24.7136 | Observer latitude (decimal degrees) |
| `observer_lon` / `OBSERVER_LON` | 46.6753 | Observer longitude (decimal degrees) |
| `observer_elev_m` / `OBSERVER_ELEVATION` | 600 | Elevation in meters |
| `timezone_name` / `OBSERVER_TZ` | `Asia/Riyadh` | IANA timezone |
| `default_min_altitude` / `DEFAULT_MIN_ALTITUDE_DEG` | 30 | Min altitude for imaging window |
| `default_packup_time` / `DEFAULT_PACKUP_TIME_LOCAL` | `01:00` | Default pack-up time HH:MM |
| `NINA_PROFILE_PATH` | — | Path to NINA profile XML (env only) |
| `NINA_SEQUENCE_DIR` | — | Output dir for NINA sequences (env only) |
| `SECRET_KEY` | `dev-secret-key-change-me` | Flask session secret (env only) |
| `DATABASE_URL` | `sqlite:///astroplanner.db` | SQLAlchemy connection string (env only) |

---

## 8. Helper Functions in `app.py`

- `get_global_config()` → `GlobalConfig` instance (creates default singleton if missing)
- `get_effective_packup_time(target)` → resolved packup time (target override > global default)
- `get_effective_min_altitude(target)` → resolved min altitude (target override > global default)
- `get_effective_calibration_config(target)` → resolved calibration counts + two_point + enabled flag
- `get_observer_location()` → `(lat, lon, elev)` tuple from GlobalConfig
- `get_palettes_with_filters()` → list of palette dicts with nested filter info
- `get_recommended_palette(target_type)` → palette name from TargetType table or fallback map
- `detect_target_type(catalog_name)` → type string using ObjectMapping DB then pattern matching
- `add_object_mapping(catalog_name, target_type_name)` → bool, saves new mapping
- `build_default_plan_json(palette_name, ...)` → plan JSON from palette template
- Template filter (via time_utils): `format_hms_from_minutes(value)` → "HH:MM:SS" string

---

## 9. Testing

- **Framework**: pytest
- **Run**: `python run_tests.py` or `pytest tests/ -v`
- **Fixture**: In-memory SQLite, seeds SHO palette with H/O/S filters
- **Coverage**: CRUD, settings, calibration suggestion/skip/manual flows (`tests/test_app.py`), migration/config tests

---

## 10. Conventions & Patterns

- **All models and routes in `app.py`** — no blueprints, no separate models package
- **Models = database schema** — columns map directly to DB columns; schema changes = model changes
- **Dark theme throughout** — `bg-dark text-light` body, `bg-secondary` cards
- **No custom CSS/JS files** — all styling via Bootstrap utilities, all JS inline in templates
- **AJAX for inline edits** — Notes and PixInsight workflow use fetch() + JSON endpoints
- **Forms use POST with redirect** — standard PRG pattern for all non-AJAX forms
- **GlobalConfig DB model** — primary config store (observer location, defaults); .env provides DB URL + secrets
- **DatabaseMigrator** — `config/migration.py` handles full bidirectional SQLite↔PostgreSQL data migration with validation and backup
- **TargetType + ObjectMapping** — auto-classification system: catalog names map to target types which recommend palettes
- **Cascade deletes** — Target deletion cascades to plans + sessions; TargetType cascades to ObjectMappings
- **Preset system** — filters/palettes/wheels can be exported/imported as JSON preset files

---
> Source: [salehram/armillarylab](https://github.com/salehram/armillarylab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
