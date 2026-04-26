## oidc-diagnositcs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

A Flask web application that acts as an OIDC client for diagnosing SSO systems (primarily Keycloak). The user logs in via SSO, then sees all claims from the ID token, access token, and UserInfo endpoint in a diagnostic UI. There is also a standalone JWT decoder that works without logging in.

## Setup & Running

```bash
# Install dependencies
pip install -r requirements.txt

# Configure
cp .env.example .env
# Edit .env with your Keycloak discovery URL, client ID, and client secret

# Run
python app.py

# Run in debug mode
FLASK_DEBUG=true python app.py
```

The app runs on port 5000 by default (`PORT` env var overrides this).

## Keycloak Client Setup

In Keycloak, create a **confidential** OpenID Connect client with:

- Valid redirect URI: `http://localhost:5000/callback`
- The client secret goes in `OIDC_CLIENT_SECRET`
- Discovery URL pattern: `http://<host>:8080/realms/<realm>/.well-known/openid-configuration`

## Architecture

Single-file Flask app ([app.py](app.py)) with Jinja2 templates:

**Routes:**

- `/` — landing page, shows config status and login button
- `/login` → `/callback` — OIDC Authorization Code flow via Authlib
- `/claims` — main diagnostic view (requires session)
- `/claims.json` — JSON export of all decoded claims
- `/decode` — standalone JWT paste-and-decode tool (no login required)
- `/logout` — clears session

**Key helpers in `app.py`:**

- `decode_jwt(token)` — base64url-decodes a JWT without signature verification
- `prepare_claims(claims_dict)` — converts a raw claims dict into typed display entries (handles timestamps, booleans, arrays, nested objects)
- `build_compare_table(...)` — merges claims from all three sources and flags mismatches

**Templates** ([templates/](templates/)):

- `base.html` — layout, all CSS (CSS variables), shared JS utilities (`copyText`, `initTabs`, `initSearch`)
- `index.html` — login landing with config status
- `claims.html` — five-tab view: ID Token / Access Token / UserInfo / Compare / Raw JWT
- `decode.html` — standalone decoder with the same claim rendering

**Interactive features on the claims page:**

- Tab switching between token sources
- Live search/filter across claim names and values
- "Mask sensitive" toggle blurs values like `sub`, `email`, `name` (useful for screenshots)
- Live expiry countdown for the access token
- Compare tab highlights claims present in multiple sources with differing values (shown with ⚠ badge)
- Raw tab shows colour-coded JWT parts (header/payload/signature)
- Copy buttons per-claim and full JSON export

**No frontend build step** — pure server-rendered Jinja2, vanilla CSS, and vanilla JS. No npm, no webpack.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tfindley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
