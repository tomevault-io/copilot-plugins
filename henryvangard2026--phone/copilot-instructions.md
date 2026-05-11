## phone

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A **Workstation Phone Management** system for tracking phones assigned to lab workstations. Implemented in multiple layers sharing a single SQLite database (`phones.db`).

## Running the Application

**CLI (interactive terminal):**
```bash
python phone.py
```

**PyQt6 GUI:**
```bash
python gui2.py
```

**FastAPI REST API** (run from the `Phone/` project root):
```bash
uvicorn fastapiapp.fastapiphone:app --reload
# API docs at http://127.0.0.1:8000/docs
```

**Flask web app** (requires FastAPI to be running for login):
```bash
flask --app flaskphone --debug run
# UI at http://localhost:5000
```

**Docker (CLI mode):**
```bash
docker build -t phone-mgr .
docker run -it -v phone_data:/data phone-mgr
```

## Architecture

All layers share the same `Phone` SQLAlchemy model and `phones.db` SQLite database defined in `phone.py`.

| File | Role |
|---|---|
| `phone.py` | Core: ORM model, DB setup, all CRUD logic (CLI + reusable functions) |
| `gui2.py` | PyQt6 GUI — imports functions from `phone.py` |
| `fastapiapp/fastapiphone.py` | FastAPI REST API with JWT auth |
| `flaskphone.py` | Flask web UI with Jinja2 templates in `templates/` |
| `kafka/kafkaphone.py` | In-progress Kafka/microservices version (not yet functional) |
| `workstation.py` | Stub — `Workstation` model (not yet implemented) |
| `user.py` | Stub — `User` model for auth (brainstorming phase) |

### Key Design Pattern: `CLI` Global Flag

`phone.py` has a module-level `CLI = True` variable. When `CLI` is `True`, functions print prompts and call `input("Press Enter to continue ...")`. The GUI must toggle this off before calling any shared function:

```python
import phone
phone.CLI = False
result = phone.viewPhones()
phone.CLI = True
```

### Database

- SQLite file: `phones.db` (gitignored)
- Phone IDs start at **1000** (seeded by the `initDB()` trick with SQLite autoincrement)
- Table: `phone` with fields: `id`, `brand`, `model`, `os`, `os_version`, `serial_number` (unique), `imei` (unique), `status`, `workstation`

### Data Validation Rules (in `phone.py`)

- **Brand:** S → SAMSUNG, A → APPLE, O → OTHER
- **OS:** A → ANDROID, I → IOS
- **OS Versions:** only `{13, 14, 15, 16, 17, 18, 26}` are valid
- **Status:** A → ACTIVE, U → UNASSIGNED, R → RETIRED
- **Workstation:** must start with `"WS"` or be exactly `"UNASSIGNED"`; phones with UNASSIGNED/RETIRED status always get workstation set to `"UNASSIGNED"`
- **Model validation:** two implementations exist — `validateModel()` (manual) and `validateModelByRE()` (regex-based); valid patterns include `S22`, `A54`, `FOLD7`, `FLIP6`, `IPHONE 14`, `IPHONE SE`

### FastAPI Auth (JWT)

The FastAPI app uses hardcoded credentials and a `SECRET_KEY`. Login via `POST /login` with form data returns a bearer token. All other endpoints require `Authorization: Bearer <token>`.

Flask's `/login` route proxies login to the FastAPI endpoint at `http://127.0.0.1:8000/login`, so FastAPI must be running for Flask login to work.

## Install Dependencies

```bash
pip install -r requirements.txt
```

Core dependencies: `SQLAlchemy>=2.0`, `fastapi`, `uvicorn`, `python-jose`, `python-multipart`, `Flask`

PyQt6 is used by `gui2.py` but is not in `requirements.txt` — install separately:
```bash
pip install PyQt6
```

---
> Source: [henryvangard2026/phone](https://github.com/henryvangard2026/phone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
