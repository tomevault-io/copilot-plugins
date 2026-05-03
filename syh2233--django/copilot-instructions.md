## django

> - `tourist_booking_system/`: Django project root (contains `manage.py`).

# Repository Guidelines

## Project Structure & Module Organization

- `tourist_booking_system/`: Django project root (contains `manage.py`).
- `tourist_booking_system/tourist_booking_system/`: Django settings/URL config (`settings.py`, `urls.py`, `wsgi.py`, `asgi.py`).
- Apps live under `tourist_booking_system/`: `accounts/`, `attractions/`, `bookings/`, `search/`, `reviews/`, `admin_dashboard/`.
- Templates: `tourist_booking_system/templates/<app>/...` (SSR via Django templates).
- Static assets: `tourist_booking_system/static/` (CSS/JS/images). Uploads: `tourist_booking_system/media/`.
- Data/scripts: `init_tourist_db.sql`, `tourist_booking_system/sql_data.sql`, `tourist_booking_system/load_data.py`, `tourist_booking_system/download_images*.py`.
- Note: there are duplicate app folders under `tourist_booking_system/tourist_booking_system/<app>/`; the active code imported by `INSTALLED_APPS` is the top-level `tourist_booking_system/<app>/`.

## Build, Test, and Development Commands

Run commands from `tourist_booking_system/`:

- Create env + install: `python3 -m venv .venv && source .venv/bin/activate && pip install -r ../requirements.txt`
- Migrations: `python3 manage.py makemigrations && python3 manage.py migrate`
- Dev server: `python3 manage.py runserver`
- Admin user: `python3 manage.py createsuperuser`
- Collect static (optional): `python3 manage.py collectstatic`

Database config is in `tourist_booking_system/tourist_booking_system/settings.py` (MySQL by default). For quick local setup, switch to the commented SQLite `DATABASES` block.

## Coding Style & Naming Conventions

- Python: 4-space indentation, PEP 8 style, `snake_case` for functions/vars, `CamelCase` for models/forms.
- Django conventions: keep routes in `<app>/urls.py`, business logic in `<app>/views.py`, templates in `templates/<app>/<page>.html`.
- Formatting/linting is not enforced in-repo; keep diffs minimal and match surrounding style.

## Testing Guidelines

- Framework: Django test runner (`unittest` style). Tests are in `<app>/tests.py`.
- Run all tests: `python3 manage.py test`; per-app: `python3 manage.py test accounts`.
- Add/adjust tests for bug fixes and critical booking/payment flows.

## Commit & Pull Request Guidelines

- Commits are short and prefixed in a lightweight Conventional Commits style (e.g., `fix: ...`, `docs: ...`, `feat: ...`).
- PRs: describe changes, include repro/verification steps, add screenshots for template/UI changes, and call out migrations or settings changes.

## Security & Configuration Tips

- Do not commit secrets (DB credentials, SMTP passwords, `SECRET_KEY`). Prefer environment variables or a gitignored local override (e.g., `local_settings.py`).
- Avoid committing generated/runtime files: `db.sqlite3`, `media/`, `__pycache__/`, `.venv/`, `.idea/`.

---
> Source: [syh2233/django-](https://github.com/syh2233/django-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
