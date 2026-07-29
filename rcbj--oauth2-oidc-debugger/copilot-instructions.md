## oauth2-oidc-debugger

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

OAuth2/OIDC Debugger — a two-service web application for testing and debugging OAuth2 and OIDC flows against real identity providers. Supports Authorization Code, Implicit, Client Credentials, Resource Owner Password, and Refresh grants, plus all three OIDC authentication flows (Authorization Code, Implicit, Hybrid).

## Architecture

The project is split into two independent Node.js services:

- **`/api/`** — Express backend (port 4000). Proxies token endpoint calls server-side and provides a `/claimdescription` endpoint with cached IANA JWT claim metadata.
- **`/client/`** — Express frontend (port 3000). Serves static HTML/JS pages and handles the OAuth2 redirect callback at `/callback`, forwarding query params to `debugger2.html`.
- **`/common/data.js`** — Shared `convertToOAuth2Format()` function used by both services to normalize grant parameters (including PKCE and custom params).

### Frontend Build

Client-side JavaScript lives in `/client/src/` and is compiled into `/client/public/js/` using **browserify** with the **envify** transform (substitutes `process.env.*` at build time). Each feature page has its own standalone bundle:

| Source | Bundle | Page |
|---|---|---|
| `debugger.js` | `debugger.js` | Authorization/Implicit initiation |
| `debugger2.js` | `debugger2.js` | Token exchange + results |
| `token_detail.js` | `token_detail.js` | JWT inspection/validation |
| `introspection.js` | `introspection.js` | Token introspection |
| `userinfo.js` | `userinfo.js` | Userinfo endpoint |
| `jwks.js` | `jwks.js` | JWKS endpoint |
| `logout.js` | `logout.js` | OIDC logout |

The browserify build runs inside Docker. There is no local build script — to rebuild bundles you must use Docker.

### Configuration

Environment-specific config files live at:
- `/api/env/{local.js,test.js,docker-tests.js}`
- `/client/src/env/{local.js,test.js,docker-tests.js}`

The active config is selected via the `CONFIG_FILE` environment variable. For local development, this is `./env/local.js`.

## Running the App

```bash
# Start all services (api + client)
CONFIG_FILE=./env/local.js docker-compose up

# Rebuild images first
CONFIG_FILE=./env/local.js docker-compose build
```

Access the app at `http://localhost:3000`.

## Running Tests

Tests use Selenium WebDriver with Chrome. A Keycloak test IdP is spun up automatically:

```bash
# Full battery of tests entirely in containers
./docker-run-tests.sh

# Tests from local shell, dependencies still in containers
./local-run-tests.sh
```

Individual test files in `/tests/`:
- `oauth2_authorization_code.js`
- `oauth2_client_credentials.js`
- `oauth2_implicit.js`
- `oidc_authorization_code.js`

There is no linting toolchain configured in this project.

## Key Implementation Notes

- **State persistence**: All user configuration (endpoints, client IDs, scopes, etc.) is stored in browser `localStorage` — passwords are intentionally excluded.
- **Token endpoint calls**: Can be made from the browser (client-side) or proxied through the API service (server-side). The UI lets users choose.
- **XSS prevention**: DOMPurify is used on the client when rendering token/claim data to the DOM.
- **SSL**: Server-side SSL certificate validation can be disabled for testing against self-signed certs.

---
> Source: [rcbj/oauth2-oidc-debugger](https://github.com/rcbj/oauth2-oidc-debugger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
