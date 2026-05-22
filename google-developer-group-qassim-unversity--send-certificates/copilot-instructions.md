## send-certificates

> - **Python >= 3.13**, managed with **uv** (`uv sync`, `uv run`)

# AGENTS.md

## Stack

- **Python >= 3.13**, managed with **uv** (`uv sync`, `uv run`)
- **FastAPI** app, ASGI entrypoint: `app.main:app`
- **Ruff** for linting and formatting (`uv run ruff check`, `uv run ruff format`)
- No test framework or type checker is configured

## Linting & formatting

Always run after making changes:

```bash
uv run ruff check --fix app/ && uv run ruff format app/
```

## Running

```bash
uv sync                                    # install deps
python run.py                              # dev server via Infisical (fetches secrets)
uv run uvicorn app.main:app --reload       # dev server without Infisical (needs secrets in env)
```

Server binds `0.0.0.0:8000`.

## System dependency

`rsvg-convert` (package `librsvg2-bin` on Debian/Ubuntu) must be on PATH. The app will fail at runtime without it.

## Secrets

Managed through **Infisical CLI** (path `/emails-backend`, env `dev` locally / `prod` in deploy). Required env vars:

- `APP_PASSWORD_KERNELTICS` — Gmail app password for `info@kerneltics.com`
- `APP_PASSWORD_GDG_QASSIM` — Gmail app password for `gdg.qu1@gmail.com`

**The app crashes on import** if both passwords are missing (`app/config.py` raises `ValueError`).

Config loaded from `.env.local` only (`.env` is ignored).

## Architecture

```
run.py                  # wrapper: infisical run -> uvicorn
app/
  config.py             # env loading, constants, email credentials
  main.py               # FastAPI app, includes routers
  routers/
    health.py           # GET /health
    emails.py           # POST /emails/certificate
    blasts.py           # POST /blasts
  services/
    certificate.py      # SVG template manipulation, rsvg-convert PNG export
    email.py            # SMTP sending (certificate emails + blast emails)
assets/
  *.svg                 # certificate templates, named {official|unofficial}-{ar|en}.svg
  email_template.html   # HTML email body with [Name] / [Event Name] placeholders
  fonts.conf            # fontconfig file passed to rsvg-convert via FONTCONFIG_FILE
```

## Deployment

Push to `main` triggers `.github/workflows/deploy.yml` — SSH to VPS, git clone, `uv sync`, restart under PM2 with Infisical in `prod` mode.

## Key details

- Certificate templates are SVG files with placeholder `<text>` elements (`{{name}}`, `{{event_name}}`, `{{date}}`, `{{gender}}`), manipulated via `xml.etree.ElementTree`.
- Arabic text gets `direction="rtl"` on SVG elements.
- Email sends retry up to 3 times with a 4-second delay between attempts.
- Blast emails use BCC; the `To` header is set to the sender address.

---
> Source: [Google-Developer-Group-Qassim-Unversity/send-certificates](https://github.com/Google-Developer-Group-Qassim-Unversity/send-certificates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
