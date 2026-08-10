## outlookemail

> This repository is a Flask multi-mailbox manager with a static web UI and a browser extension.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Flask multi-mailbox manager with a static web UI and a browser extension.

- `web_outlook_app.py` is the main compatibility entrypoint.
- `outlook_web/` contains segmented application modules and runtime helpers.
- `templates/` and `static/` hold the server-rendered HTML, CSS, and JavaScript.
- `browser-extension/` contains the Chrome/Edge Manifest V3 extension.
- `tests/` contains pytest tests named `test_*.py`.
- `docs/`, `README.md`, `RELEASE.md`, and `CHANGELOG.md` hold user and release docs.

## Build, Test, and Development Commands

Create an environment and install dependencies:

```bash
python -m venv venv
pip install -r requirements.txt
```

Run the app locally:

```bash
python web_outlook_app.py
```

The app listens on `http://localhost:5000` by default. Set `SECRET_KEY` for stable sessions and encryption. For Docker:

```bash
docker build -t outlookemail .
docker run -p 5000:5000 -e SECRET_KEY=your-secret-key outlookemail
```

Run tests with:

```bash
python -m pytest
```

Focused checks for current shared-mailbox and API behavior:

```bash
python -m pytest tests/test_temp_email_share.py tests/test_account_share.py tests/test_external_verification_code_api.py tests/test_env_loading.py -q -p no:cacheprovider
python -m compileall web_outlook_app.py outlook_web
```

## Coding Style & Naming Conventions

Use Python 3.9+ compatible code. Follow the existing style: 4-space indentation, descriptive snake_case, and concise helpers near the behavior they support. Keep route, scheduler, storage, and provider logic in the matching `outlook_web/segments/` file unless a shared helper belongs in `outlook_web/`.

Frontend code is plain JavaScript and CSS. Match existing DOM utilities, class naming, and four-column layout behavior.

## Testing Guidelines

Add focused pytest coverage for backend behavior changes, especially mailbox import, IMAP/Graph fallback, retention, Docker update, and runtime behavior. Name files `tests/test_<feature>.py` and functions `test_<expected_behavior>`. Mock network, Docker, and mail-provider calls.

When testing environment loading, set env vars before importing `web_outlook_app`; for `.env` bootstrap behavior, prefer a subprocess with `OUTLOOK_EMAIL_ENV_FILE` so module import state cannot leak between tests. Real OS environment variables must continue to take precedence over `.env` values.

For public share or external API tests, use Flask test clients and mocked mail-provider helpers. Do not hit real Outlook/IMAP providers from pytest.

After each complete implementation, run a real local HTTP end-to-end smoke test when the change affects runtime behavior, external APIs, sharing, authentication, database migrations, Docker/runtime startup, or frontend-to-backend flows. This is in addition to pytest and compile checks: start or reuse `python web_outlook_app.py`, call the running service on `http://127.0.0.1:5000`, use the configured external API key, and verify the behavior against the real local SQLite database. For the external mailbox API, use:

```bash
python scripts/e2e_external_api_smoke.py --api-key <external-api-key> --group-ids 1,2,49,50 --claim-group-id 49
```

If the local `.env` contains `EXTERNAL_API_KEY`, the `--api-key` argument can be omitted:

```bash
python scripts/e2e_external_api_smoke.py --group-ids 1,2,49,50 --claim-group-id 49
```

The smoke test should perform real HTTP requests such as `/api/external/accounts` and a safe `claim -> release` cycle. Do not hard-code real API keys in tracked files; pass them via command line, environment variables such as `OUTLOOK_EXTERNAL_API_KEY`, or the untracked local `.env`.

## Runtime, Sharing, and External API Notes

`web_outlook_app.py` loads a project `.env` via `outlook_web.runtime.load_environment_file()` before segmented bootstrap. Keep this behavior early in the entrypoint, and do not override real environment variables with `.env` values.

Mailbox sharing uses both `temp_email_shares` and `account_shares`; tokens must be unique across both tables. `/shared/<token>` and `/api/shared/<token>/*` support both `share_type=temp_email` and `share_type=account`. Public share responses must stay read-only and must not expose account credentials, proxy settings, forwarding settings, raw MIME, or attachment download capability. HTML body rendering on the public page must continue to use `DOMPurify.sanitize`.

The external API uses `api_key_required` for `/api/external/*`. `GET /api/external/verification-code` is a code-only endpoint: it may read candidate message details server-side, but it must not return full bodies or credentials. Its default extraction targets 6-digit verification codes and should prefer verification-code context and code-block HTML over arbitrary numbers in unrelated messages.

## Commit & Pull Request Guidelines

Recent commits use short messages such as `feat: support multi-keyword account search`, Chinese summaries, and `release: 2.0.57`. Keep commits focused and mention user-visible behavior.

Pull requests should include a clear description, test results, linked issues when applicable, and screenshots for UI or extension changes. Update `README.md`, `docs/`, `CHANGELOG.md`, or `VERSION` when behavior, APIs, deployment, or releases change.

## Security & Configuration Tips

Never commit real mailbox credentials, refresh tokens, API keys, WebDAV passwords, or production `SECRET_KEY` values. Use `.env.example` for examples. Keep Gunicorn single-worker because scheduler and SSE state are in-process.

## Agent-Specific Instructions

On Windows PowerShell 5.1, read Chinese or UTF-8 files with `Get-Content -Raw -Encoding UTF8 <path>`. When using .NET APIs, decode bytes with `[System.Text.Encoding]::UTF8.GetString([System.IO.File]::ReadAllBytes(<path>))`. Do not diagnose mojibake from plain `Get-Content` output.

---
> Source: [ShirlyTaylor73/outlookEmail](https://github.com/ShirlyTaylor73/outlookEmail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
