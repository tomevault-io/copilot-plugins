## mcp-deploy

> SaaS platform that lets an admin catalog, version, and deploy MCP servers to an enterprise fleet. End users run a CLI agent (`mcp-agent`) that auto-installs/updates MCPs and injects credentials (E2E encrypted).

# MCP Deploy — Agent guide

## Project overview

SaaS platform that lets an admin catalog, version, and deploy MCP servers to an enterprise fleet. End users run a CLI agent (`mcp-agent`) that auto-installs/updates MCPs and injects credentials (E2E encrypted).

**Two repos in one tree:**
- `apps/` — Django backend (API + admin)
- `client/` — Python CLI (`mcp-agent`) installable via pip

## Quick start

```bash
source venv/bin/activate
make migrate     # django-admin makemigrations && migrate
make dev         # python manage.py runserver
```

Settings: `config/settings/dev.py` (SQLite) / `config/settings/prod.py` (PostgreSQL, used on Render). Use `DJANGO_SETTINGS_MODULE=config.settings.dev`.

Seed test data: `python manage.py seed_test_data`

## Architecture

### Django apps (`apps/`)
| App | Role |
|-----|------|
| `accounts` | User, API tokens, EC public key, key challenge (proof-of-possession) |
| `catalog` | MCPServer, Scope, MCPScope, MCPServerVersion, SystemConfig |
| `access` | MCPServerAccess (user↔MCP), UserScopeValue, GroupScopeValue (E2E encrypted) |
| `audit` | AuditLog (all admin actions + client events) |
| `monitoring` | ClientStatus, DeploymentStatus, DeploymentError |
| `dashboard` | Custom views outside Django Admin (Chart.js, no React) |

### Key API endpoints (Django Ninja, all under `/api/v1/`)

| Endpoint | Auth | Purpose |
|----------|------|---------|
| `GET /sync/` | Token | List MCPs + encrypted credentials for this user |
| `POST /ack/{slug}/` | Token | Confirm MCP installed |
| `POST /report-error/` | Token | Report client-side install error |
| `POST /heartbeat/` | Token | Update client health status |
| `GET /accounts/me/` | Token | Current user info |
| `GET /accounts/challenge/` | Token | Get challenge for key registration (ECDSA proof) |
| `POST /accounts/public-key/` | Token | Register public key + signed challenge |
| `GET /accounts/client-version/` | Public | Latest CLI version |
| `POST /re-encrypt/` | Token | Re-encrypt credential after key rotation |

Auth middleware (`apps/middleware.py`): Bearer token → SHA256 hash → lookup `UserAPIToken`. Token stored hashed, never in plaintext.

### Authentication flow
1. Admin creates user → gets one-time raw token (`mcp_xxx`)
2. User runs `mcp-agent setup` → generates EC key pair → server challenge → signs with ECDSA → server stores public key
3. All subsequent API calls use Bearer token

### E2E encryption
- **Client**: `identity.key.enc` (AES-256-GCM, key in OS keychain)
- **Server**: `encrypt_for_user(user.public_key, plaintext)` via ECIES (ECDH + AES-GCM)
- **Server never sees plaintext**: admin enters value → encrypted immediately with user's public key
- **Recovery**: `recovery.txt` generated at setup, printed once
- **Rotation**: `mcp-agent rekey` generates new keys, re-encrypts credentials via `/re-encrypt/`

### Client CLI (`mcp-agent`)
| Command | Purpose |
|---------|---------|
| `setup` | Configure server URL + token + generate identity |
| `sync` | Download MCPs, decrypt E2E blobs, write host configs |
| `start/stop` | Daemon mode (polling 10 min) |
| `rekey` | Rotate EC keys + re-encrypt all credentials |
| `get-creds` | Decrypt and print credentials (used by wrappers) |
| `status` | Show installed MCPs |

Hosts supported: Claude Desktop (`~/Library/Application Support/Claude/`), VS Code/Claude Code (`~/.mcp.json`), Cursor (`~/.cursor/mcp.json`).

## Secrets / env vars

**Development** (`.env`):
```
DJANGO_SETTINGS_MODULE=config.settings.dev
SECRET_KEY=dev-secret-key-not-for-production
FERNET_KEY=<generated once: Fernet.generate_key()>
```

**Production** (set in Render dashboard):
| Var | Required | Notes |
|-----|----------|-------|
| `SECRET_KEY` | Yes | Render generates |
| `ADMIN_PASSWORD` | For first deploy | Sets admin password via `setup_admin` command |
| `SEED_TEST_DATA` | For test | Set `=1` + `MCP_PATH` to seed test data at deploy |
| `MCP_PATH` | With `SEED_TEST_DATA` | Absolute path to `weather_mcp.py` on target Mac |
| `FERNET_KEY` | No | Auto-generated at startup if unset |

Render config: `render.yaml` + `render_start.sh`. Free plan (web + PostgreSQL). No Redis/Celery.

## Key commands

```bash
# Backend
make migrate             # makemigrations + migrate
make dev                 # runserver on :8000
make admin               # createsuperuser
pip install -r requirements/dev.txt

# Client (install from source)
pip install -e client/
mcp-agent setup --server <URL> --token <TOKEN>
mcp-agent sync

# Test
python tests/test_api_server.py &   # local validation API on :8766
```

## Deployment

Render Blueprint deploy from `render.yaml`. Push to GitHub `main` branch → auto-deploy. To trigger re-seed: toggle `SEED_TEST_DATA` env var.

## Important constraints

- **No React/JS framework**: admin UI uses Django Templates + Chart.js CDN
- **No Redis/Celery**: polling (10 min), no push/WebSocket for MVP
- **`allowed_commands`** SystemConfig controls which commands MCPs can run (defaults: `npx, uvx, python3, python, node, /bin/bash, /bin/sh`)
- **Tokens are hashed** (SHA256) in DB, never stored in plaintext
- **Credentials are E2E encrypted**: server cannot read them. If key is lost, recovery key needed or admin must re-enter credentials
- **Private key** (`identity.key.enc`) is encrypted with a key stored in the OS keychain. If keychain is reset, the identity is lost
- **Python 3.14**: Django admin custom methods in `list_display` must return strings, not format_html objects (compat issue)

---
> Source: [maxencebb/mcp-deploy](https://github.com/maxencebb/mcp-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
