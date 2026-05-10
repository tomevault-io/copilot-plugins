## breach-tracker

> Internal tool for Wiom to detect and manage partner breach cases across 3 types:

# Breach Tracker — CLAUDE.md

## Project Overview

Internal tool for Wiom to detect and manage partner breach cases across 3 types:
- **B1** (Fundamental Principle 1) — Disintermediation (putting Wiom connections on other ISPs)
- **B2** (Fundamental Principle 2) — Extra Cash Collection from customers
- **B4** (Fundamental Principle 4) — Router/Netbox Misuse

## Architecture

### Backend
- **Flask** app (`app.py`) with Basic HTTP auth (`APP_USER`/`APP_PASS`, default: wiom/wiom2026)
- **Railway** deployment via Gunicorn + Gevent
- APScheduler: auto-sync every 2 min, Slack digest every 30 min

### Data Layer
- **B2**: Google Sheets via `sheets_db.py` (Sheet ID in `BREACH2_SHEET_ID` env var, service account auth)
- **B1/B4**: SQLite via `db.py`
- Partner emails: fetched from Google Sheets "Email_dump" tab via `google_sheets.py`

### Two Frontends (keep in sync)
- `templates/index.html` — served by Flask on Railway
- `docs/index.html` — GitHub Pages static site, uses `apiGet()`/`apiPost()` to call Railway API

### Email
- `email_sender.py` — FP1, FP2, FP4 templates with Hindi/English/Both
- Gmail OAuth2 primary (works on Railway), SMTP fallback
- FP2 has custom inline body (not generic proof block pattern)

## Key Files

| File | Purpose |
|------|---------|
| `app.py` | Flask routes, auth, scheduler, sync logic |
| `sheets_db.py` | Google Sheets CRUD for B2 cases (range `Cases!A2:W10000`) |
| `db.py` | SQLite for B1/B4 cases |
| `email_sender.py` | Email templates (FP1/FP2/FP4) + Gmail OAuth2/SMTP sending |
| `actions.py` | CSV/Excel generation (refund CSV, penalty xlsx, partner comms) |
| `google_sheets.py` | Read partner emails, disintermediation cases, FP4 cases from Sheets |
| `kapture.py` | KaptureRM CRM API client for ticket enrichment |
| `metabase.py` | Metabase SQL queries for breach detection |
| `config.py` | Settings from `settings.json` or env vars |
| `templates/index.html` | Railway frontend |
| `docs/index.html` | GitHub Pages frontend (mirror, uses apiGet/apiPost) |

## B2 Workflow States

```
detected → customer_refunded → customer_comms → partner_penalty
```

- **detected**: Synced from Metabase, enriched from Kapture (extra_amount, technician_name)
- **customer_refunded**: After refund CSV upload (matched by customer_mobile)
- **customer_comms**: After calling customer, confirming refund (has comms_notes field)
- **partner_penalty**: After penalty xlsx upload (matched by partner AccountId)

## Environment Variables

### Required on Railway
- `GOOGLE_SERVICE_ACCOUNT_JSON` — Service account JSON (secret)
- `GMAIL_CLIENT_ID`, `GMAIL_CLIENT_SECRET`, `GMAIL_REFRESH_TOKEN` — Gmail OAuth2
- `APP_USER`, `APP_PASS` — Basic auth credentials

### Optional / Have Defaults
- `BREACH2_SHEET_ID` — Google Sheet for B2 (default in sheets_db.py)
- `METABASE_URL`, `METABASE_API_KEY` — Breach detection queries
- `KAPTURE_AUTH_HEADER`, `KAPTURE_COOKIE` — CRM enrichment
- `SLACK_WEBHOOK_URL` — Digest notifications

## Conventions

- Service account: `kapture-bot@kapture-488314.iam.gserviceaccount.com`
- Partner IDs from Sheets may have `.0` suffix — always normalize with `int(float(pid))`
- `ticket_added_time_ist` = actual case creation date (ISO format with T separator)
- `detected_at` = when system synced the case (NOT the real case date)
- Penalty xlsx columns: `AccountId` (integer), `Amount` (negative), `Remark`
- Penalty result xlsx has additional columns: `Process Status`, `Reason`, `AddedTime`, `transactionId`

## Critical: Two Frontends Must Stay in Sync

`templates/index.html` (Railway) and `docs/index.html` (GitHub Pages) are independent copies — NOT generated from a shared source. **Every change to one MUST be mirrored to the other.**

### Sync checklist when editing frontend code:
1. **JS functions**: If you add/modify/remove a function in one file, do the same in the other
2. **HTML structure**: If you add/remove a div/element (e.g. a sub-tab), update ALL JS references in both files
3. **DOM IDs**: If you remove an element, grep BOTH files for its ID — dead references cause silent JS errors that break entire functions
4. **API patterns differ**: templates uses `api()` (returns `{}` on error), docs uses `apiGet()`/`apiPost()` (returns `null` on error) — handle both
5. **After editing, verify**: grep both files for any ID/function referenced in your change

### Past bugs from sync failures:
- `switchB2Sub()` in docs referenced `#b2sub-visibility` after it was removed → broke entire Comms tab
- `api()` returns `{}` on error but code assumed array → needed `Array.isArray()` safety check in templates

## Commands

```bash
# Run locally
python3 app.py  # starts on port 5050

# Production
gunicorn app:app --worker-class gevent --bind 0.0.0.0:$PORT --timeout 120
```

---
> Source: [shivanksood-prog/breach_tracker](https://github.com/shivanksood-prog/breach_tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
