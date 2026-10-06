## intellikeep

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IntelliKeep is a custom Home Assistant integration for household task management. It provides a full-stack solution: Python backend (HA integration) + TypeScript/Lit frontend (panel sidebar + Lovelace card).

## Commands

### Testing (Python backend)

```bash
# Run all tests via Docker (no local Python required)
./run-tests.sh

# Run a specific test file
./run-tests.sh tests/test_task_manager.py

# Filter by test name
./run-tests.sh -k test_complete_recurring

# Run locally (requires Python 3.12+ and dependencies)
pip install -r requirements_test.txt
pytest tests/ -v
```

### Frontend Build

```bash
# Build both panel and Lovelace card via Docker
./build-frontend.sh

# Build only one
./build-frontend.sh card
./build-frontend.sh panel

# Build locally (Node 20+ required)
cd panel && npm install && npm run build
cd lovelace-card && npm install && npm run build
```

Built artifacts land in `custom_components/intellikeep/frontend/` and are **committed to git** (HACS serves them from the repo) — after any frontend change, rebuild and commit the bundles.

To deploy to a running Docker-based HA instance:

```bash
docker cp custom_components/intellikeep/ homeassistant:/config/custom_components/
docker restart homeassistant
```

## Architecture

### Backend (`custom_components/intellikeep/`)

The integration follows a layered pattern:

```
WebSocket API / Services (user actions)
  ↓
TaskManager (business logic)
  ↓
Storage (JSON persistence via HA Store API)
  ↓
Coordinator (5-min refresh → sensor updates)
  ↓
NotificationManager (hourly checks → persistent notifications / push)
```

Key modules:
- **`models.py`** — Data classes (`Task`, `TaskExecution`, `TaskNote`, `TaskActivity`) and enums (`TaskFrequency`, `TaskPriority`, `TaskStatus`, `TaskActivityType`). Task status (`PENDING`, `DUE`, `OVERDUE`) is calculated in real-time from `due_date`; it is not stored.
- **`task_manager.py`** — All business logic: CRUD, status calculation, recurring task scheduling, note management.
- **`storage.py`** — Wraps HA's `Store` API. Maintains a sequential `task_number` counter that never resets (persists across deletions).
- **`coordinator.py`** — `DataUpdateCoordinator` refreshing every 5 minutes for sensor entities.
- **`services.py`** — Registers HA services (`create_task`, `complete_task`, `reopen_task`, `update_task`, `delete_task`, `add_task_note`, `delete_task_note`, `load_sample_data`, `delete_all_data`).
- **`websocket_api.py`** — WebSocket commands used by the panel and card (`get_tasks`, `get_task`, `subscribe`, `get_version`).
- **`notifications.py`** — Hourly background task; deduplicates within a session (resets on HA restart).
- **`sensor.py`** — Three sensor entities: `tasks_due_today`, `tasks_overdue`, `next_due_task`.
- **`config_flow.py`** — Single-instance config flow. Options: `instance_name`, `notify_days_before_default`, `notification_service`.
- **`runtime_data.py`** — Type-safe dataclass bundling runtime objects stored on the config entry.
- **`seed.py`** — Generates sample tasks for demo via `load_sample_data` service.

Recurring task model: each occurrence links to the root via `previous_task_id` (flat-star pattern, no tree). Completing a recurring task auto-creates the next occurrence.

### Frontend (`panel/` and `lovelace-card/`)

Both use Rollup + TypeScript + Lit web components. The panel is the full sidebar UI; the card is a compact dashboard widget.

- `panel/src/api.ts` — WebSocket client wrapping HA's connection API
- `panel/src/types.ts` — TypeScript interfaces mirroring backend models
- `panel/src/views/` — Page-level components (task list, task form, calendar, history, settings)
- `panel/src/components/` — Reusable UI elements

The frontend is decoupled from business logic; it communicates exclusively through the WebSocket API.

### Translations

Two separate systems: the backend uses `strings.json` + `translations/{en,es,pt}.json` (config flow, services, sensor names), while the panel has its own translation table in `panel/src/translations.ts`. User-visible string changes usually touch both.

## Key Implementation Details

- **Single instance only** — `single_config_entry: true` in `manifest.json`; only one IntelliKeep per HA installation.
- **No external Python dependencies** — Uses only HA core APIs (`voluptuous`, `homeassistant.helpers`).
- **Requires HA 2026.1.0+**
- **Version is duplicated** — bump `manifest.json`, `const.py` (`VERSION`, served to the frontend via the `get_version` WebSocket command), and `panel/package.json` together when releasing; update `CHANGELOG.md`.
- **HA bus events** — `intellikeep_task_notification` is fired by the notification manager (documented automation hook). `intellikeep_task_updated` is fired by `TaskManager` on every mutation (`created`, `updated`, `completed`, `reopened`, `deleted`, `note_added`, `note_deleted`). Panel/card live updates flow through coordinator listeners via the `subscribe` WebSocket command, not through these events.
- **CI/CD** — GitHub Actions runs pytest on Python 3.12 & 3.13 (push to `main`/`dev` and PRs), plus HACS validation, a frontend build, and `hassfest` on `main` pushes/PRs.

---
> Source: [henriquekraemer/intellikeep](https://github.com/henriquekraemer/intellikeep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
