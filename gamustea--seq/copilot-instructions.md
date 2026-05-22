## seq

> docker compose --profile dev up -d

# SeQ — Agent Guide

## Quick Start

```bash
docker compose --profile dev up -d
cd API && python run.py
```

## Ports

| Service | Port | Note |
|---|---|---|
| PostgreSQL | **15432** | Not 5432 |
| OpenVAS | 9390 | ~15min first start |
| Ollama | 11434 | |
| API | 5000 | |

## Modules

- `API/src/modules/sentinel/` — Nmap, Nikto, OpenVAS + PDF/IA reports
- `API/src/modules/aegis/` — Awareness pills (Ollama)
- `API/src/modules/acheron/` — Encrypted secret vault

Entry: `API/run.py`

## Auth

```
Authorization: Bearer <access_token>
```

- Get token: `POST /oauth/token` with `{"grantType": "password", "username": "root", "password": "admin"}`
- Refresh: `grantType: refresh_token`

## Dev Commands

```bash
cd API && pytest
cd API && flake8 .
cd API && mypy .
cd API && black --check .
```

## Documentation

Push to `main` → docs generated in `docs/py/` → push to `docs` branch → GitHub Pages.

## Config

- `API/.env` — connects to `localhost:15432`
- `API/SecOpsConfig.json` — AI prompts

## Ignore & Security

- `.env` has credentials — never commit
- `API/src/data/` is gitignored
- `API/.gitignore` excludes scan outputs

---
> Source: [gamustea/SeQ](https://github.com/gamustea/SeQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
