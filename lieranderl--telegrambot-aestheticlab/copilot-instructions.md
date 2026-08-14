## telegrambot-aestheticlab

> - Use **uv** only: `uv sync`, `uv run ...`; Python is pinned to `==3.12.*`

# Agent Instructions

## Package Manager
- Use **uv** only: `uv sync`, `uv run ...`; Python is pinned to `==3.12.*`
- Keep `uv.lock` synced when dependencies change

## Project Shape
- App factories: `src/app.py`
- Public entrypoint: `src.main:app`; admin entrypoint: `src.admin_main:app`
- Environment settings: `src/config.py`
- Routes: `src/routes/`; integrations: `src/gateways/`; orchestration: `src/services/`
- Models/errors/helpers: `src/models.py`, `src/errors.py`, `src/utils/`

## Local Commands
| Task | Command |
|------|---------|
| Install | `uv sync` |
| Run public app | `uv run uvicorn src.main:app --host 0.0.0.0 --port 8080 --reload` |
| Run admin app | `uv run uvicorn src.admin_main:app --host 0.0.0.0 --port 8081 --reload` |
| Full tests | `uv run python -m unittest discover -s tests` |
| Coverage | `uv run coverage run -m unittest discover -s tests && uv run coverage report -m` |

## File-Scoped Commands
| Task | Command |
|------|---------|
| Single test file | `uv run python -m unittest tests/test_config.py` |
| Single test case | `uv run python -m unittest tests.test_config.SettingsTests.test_from_env_raises_for_missing_values` |

## Testing
- Maintain the coverage threshold enforced in `pyproject.toml`
- Keep tests in `tests/`; mock Google Calendar, Telegram, HTTP, and Firestore calls
- Unit tests must not require live Google Cloud, Telegram, or network access

## Runtime Contracts
- Config comes only from environment variables; keep names aligned with `README.md`
- Required env: `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`, `WEBHOOK_URL`, `CALENDAR_IDS`
- Optional env: `STATE_COLLECTION_PREFIX`, `RENEWAL_LEAD_MINUTES`, `GOOGLE_CLOUD_PROJECT`, `GCP_PROJECT`
- Keep secrets out of source, logs, tests, and docs examples
- Preserve Google webhook validation for channel ID, token, resource ID, and `sync` handshake notifications
- Admin routes are protected by Cloud Run IAM only. Do not reintroduce app-level shared-token auth for admin endpoints.
- Google Calendar watch renewal is automated by Cloud Scheduler calling private admin `/admin/renew` with OIDC.
- The renewal scheduler uses a dedicated `calendar-telegram-renewer` service account with only `roles/run.invoker` on the admin Cloud Run service.
- Firestore state collections are part of the service contract:
  - `{prefix}_calendar_states/{sha1(calendar_id)}`: sync token and metadata
  - `{prefix}_channels/{channel_id}`: channel metadata, token, and expiration
  - `{prefix}_deliveries/{sha1(calendar_id|event_id|event_version)}`: delivery de-duplication marker

## Code Conventions
- Keep route handlers thin and orchestration in services
- Keep gateways as integration boundaries; do not call Google Calendar, Telegram, or Firestore directly from routes
- Use typed dataclasses/models for shared data; prefer `AppServices` injection and app state for tests
- Avoid new frameworks, background schedulers, or deployment tooling unless requested

## Deployment
- Production GCP project: `nail-lab-449417`
- CI checks: `.github/workflows/_checks.yml`, `.github/workflows/ci.yml`
- Production deployment: `.github/workflows/deploy.yml` builds one immutable image and deploys both public and admin Cloud Run services.
- Admin-only manual fallback: `.github/workflows/deploy-admin.yml`
- Public service exposes only `/health` and `/webhook`; admin routes live only in `src.admin_main:app`

## Commit Attribution
AI commits MUST include:
```
Co-Authored-By: (the agent model's name and attribution byline)
```

---
> Source: [lieranderl/telegrambot-aestheticlab](https://github.com/lieranderl/telegrambot-aestheticlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
