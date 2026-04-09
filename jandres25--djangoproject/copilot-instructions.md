## djangoproject

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Django 4.2.3 web application with three apps, using Python 3.12+ inside a virtualenv (`proyectodjango/`). The Django project root is `myproject/` (contains `manage.py`).

## Commands

All commands must be run from `myproject/` after activating the virtualenv:

```bash
# Activate virtual environment (from repo root)
source proyectodjango/bin/activate

# Run dev server
cd myproject
python manage.py runserver

# Apply migrations
python manage.py migrate

# Create new migrations after model changes
python manage.py makemigrations

# Run tests
python manage.py test

# Run tests for a single app
python manage.py test helloUPDS
python manage.py test portfolio
python manage.py test api

# Open Django shell
python manage.py shell
```

## Architecture

Three Django apps mounted at:

| App         | URL prefix    | Purpose                                              |
| ----------- | ------------- | ---------------------------------------------------- |
| `helloUPDS` | `/`           | Home/landing page                                    |
| `portfolio` | `/portfolio/` | Portfolio projects from SQLite DB                    |
| `api`       | `/api/`       | Displays paginated data from external FreeToGame API |

### Data flow

- **portfolio**: `Proyecto` model (titulo, descripcion, tecnologias, link, imagen) stored in `db.sqlite3`. Views: `projectIndex` (list all), `projectDetail` (single by PK).
- **api**: No local model — fetches live data from `https://www.freetogame.com/api/games` via `requests`, paginates results (10/page) and renders them.
- **helloUPDS**: Static landing page, no model.

### Templates

Each app has its own `templates/` directory. The global `TEMPLATES['DIRS']` also points to `helloUPDS/templates/`, so `base.html` is the shared base template used by all apps.

- `base.html` — full HTML structure with navbar, footer, Bootstrap 4.6.2, jQuery 3.7.1, Popper.js 1.16.1
- `baseapi.html` — extends `base.html`, adds DataTables 1.13.6 (Bootstrap 4 integration)
- Child templates use `{% block page_content %}` and optionally `{% block extra_css %}` / `{% block extra_js %}`

### Database

SQLite at `myproject/db.sqlite3`. Only `portfolio` has a migration (`0001_initial`). The `api` app has no models.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jandres25)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jandres25)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
