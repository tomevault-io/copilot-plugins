## bulk-email-sender

> - Modular Flask app using Application Factory + Blueprints (auth, main, api).

# Cursor Project Rules for Bulk Email Sender

## Context
- Modular Flask app using Application Factory + Blueprints (auth, main, api).
- Service layer: app/services/excel_service.py and app/services/email_service.py.
- Dedicated clients: app/clients/mailgun.py for Mailgun API integration.
- SQLAlchemy 2.0-style models with typing in app/models.py.
- Tailwind/Quill front-end, all JS in static/js/app.js; minimal CSS in static/css/app.css.
- Excel upload (.xlsx/.xls) parsed with Pandas/OpenPyXL; NaN normalized to None in DB; None rendered as empty strings in previews.
- Template import supports .txt, .docx, .pdf (python-docx, PyMuPDF).
- 3-column responsive UI: left tree of files/columns, middle context (editable rows/column info), right template editor + preview.
- Bulk sending integrated with Mailgun API.

## Conventions
- Follow PEP 8 and use type hints across Python code. Keep imports sorted (isort) and formatting via Black.
- For JS, follow ESLint recommended rules and format with Prettier.
- Preserve per-user authorization checks on all data access.
- Maintain Tailwind utility approach in templates; keep components responsive. Prefer extracted CSS over inline <style>.
- For template tags, placeholders must be in the form {ColumnName} and match Excel headers exactly.

## Dependencies & Tooling
- Primary dependency manifest: pyproject.toml managed with uv. Lockfile: uv.lock.
- If you need a requirements.txt, export from uv: `uv export --format requirements.txt --output-file requirements.txt`.
- Python 3.13+. Prefer virtual env/uv virtualized runs.
- Linters/formatters:
  - Python: Black, isort, Ruff (configured in pyproject.toml)
  - JS: ESLint + Prettier (.eslintrc.json, .prettierrc.json)
- Editor settings standardized via .editorconfig

## Running locally
- Recommended: `uv sync` then `uv run python app.py`.
- Alternative: `pip install -r requirements.txt` then `python app.py`.
- For production CSS, install Tailwind via npm and build a minimized CSS bundle; CDN usage is dev-only.

## Data & Files
- Do not commit `instance/database.db` or the `uploads/` contents; treat as runtime artifacts.
- Ensure uploads path exists and is user-writable. Max upload size 16MB (configurable).

## Security & Privacy
- Do not hardcode secrets for production. SECRET_KEY in app.py is dev-only; document env-based overrides.
- Mailgun credentials (API key, domain) are loaded from `.env` via `python-dotenv`.
- Keep auth-required on all non-auth endpoints; never bypass access checks.
- Consider adding CSRF if exposing JSON endpoints publicly.

## When changing behavior
- Update README.md with new features/endpoints.
- Keep placeholders behavior (None -> empty string) intact unless explicitly changed.
- Validate large Excel handling and column name edge cases.

## Testing notes
- If adding tests, set seeds when randomness is involved.
- Prefer small test spreadsheets checked into a `tests/fixtures/` folder.

## Non-goals (unless requested)
- Bulk sending/mailing, SMTP integration, or rate limiting.
- Multi-template management. (Currently a single template per user.)
- Deleting files or admin dashboards.

## Review checklist before committing
- [ ] Authorization enforced per current_user for all resource reads/writes
- [ ] Placeholders replacement safe for None and non-string values
- [ ] Large files handled with size guard and memory-safe parsing
- [ ] UI remains responsive on small screens
- [ ] README and steering files updated if endpoints/flows changed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ultrasardine) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
