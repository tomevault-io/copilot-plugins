## jarvis-auth

> The identity backbone. Three independent auth modes (user JWT, app-to-app credentials, node credentials), full household multi-tenancy, and an admin surface for credential lifecycle. Everything downstream of identity flows through here.

# jarvis-auth

The identity backbone. Three independent auth modes (user JWT, app-to-app credentials, node credentials), full household multi-tenancy, and an admin surface for credential lifecycle. Everything downstream of identity flows through here.

---

## What this service is (and isn't)

| Mode | Who | Header / Token | Where validated |
|---|---|---|---|
| **User JWT** | Humans (admin UI, web, mobile) | `Authorization: Bearer <jwt>` | **Locally** by each service using shared `AUTH_SECRET_KEY` — no round-trip to auth |
| **App-to-app** | Services calling each other | `X-Jarvis-App-Id` + `X-Jarvis-App-Key` | Round-trip to `/internal/validate-app` (or `/internal/app-ping`) |
| **Node** | Pi Zero nodes | `X-API-Key: node_id:node_key` (service-side header convention) | Round-trip to `/internal/validate-node` (which also checks per-service access) |

**Not** a:
- Permissions / RBAC engine for application data (only household-level role hierarchy: MEMBER < POWER_USER < ADMIN, used for household-scoped actions)
- Session store — JWTs are self-contained; refresh tokens are the only server-side session state
- Identity provider for third-party SSO (no OIDC/OAuth flows; local user/password only)

---

## Quick Reference

```bash
# Local dev (Docker default port 7701)
poetry install
cp env.template .env  # set AUTH_SECRET_KEY, DATABASE_URL, JARVIS_AUTH_ADMIN_TOKEN at minimum
alembic upgrade head
docker-compose -f docker-compose.dev.yaml up --build
# or local: poetry run uvicorn jarvis_auth.app.main:app --reload --port 7701

# Test
poetry run pytest
```

---

## Dependency graph

**Upstream (jarvis-auth depends on):**
- **PostgreSQL** (required) — users, refresh tokens, app clients, nodes, households, invites, settings
- **jarvis-logs** (optional, port 7702) — centralized logging; falls back to console on failure
- **jarvis-config-service** (optional, port 7700) — service discovery. Auth makes no outbound calls on the hot path, with **one exception**: `DELETE /auth/me` (account deletion) fans out a best-effort user-data purge to `jarvis-command-center` and `jarvis-notifications` (`DELETE /api/v0/me/data`, forwarding the user's Bearer token). Their URLs are resolved from config-service, or via `JARVIS_COMMAND_CENTER_URL` / `JARVIS_NOTIFICATIONS_URL` overrides. The purge is tolerant-blocking: unreachable services are skipped (best effort), but a downstream 5xx aborts the deletion (502) before any local data is touched.

**Downstream (depends on jarvis-auth):**
- **All services that validate JWTs** — share `AUTH_SECRET_KEY`, validate locally (no network call)
- **All services that need app-to-app auth** — round-trip to `/internal/app-ping` or `/internal/validate-app`
- **All services that accept nodes** (command-center, whisper, tts, logs) — round-trip to `/internal/validate-node`
- **jarvis-command-center** — also calls `/internal/validate-household-access`, `/internal/validate-node-household`, `/internal/users/batch` (speaker resolution)
- **jarvis-config-service** — uses `JARVIS_AUTH_ADMIN_TOKEN` to call `/admin/app-clients` during first-boot bootstrap
- **Browser clients**: `jarvis-admin`, `jarvis-web`, `jarvis-node-mobile` — hit `/auth/login` etc. directly (CORS allow-listed)

**Impact if down:**
- New user logins fail
- App-to-app authentication fails (anything calling `/internal/validate-app` returns 5xx) → cascade failure across the stack
- Node authentication fails → nodes lose access
- **Existing user JWTs keep working until expiry** (30min default) because validation is local-only

---

## Lifecycle / common operations

### 1. First-boot superuser bootstrap

```
GET  /auth/setup-status        → {"needs_setup": true} if no superusers
POST /auth/setup                → creates first user as superuser + admin of a new "My Home" household
```

The installer calls this once. After that, the endpoint refuses (409) — there's no other path to creating a superuser today (no env-var seeding). **Don't add one without a real reason** — restricting superuser creation to a one-shot bootstrap is a deliberate safety measure.

### 2. User register / login / refresh

`POST /auth/register` (or `/auth/login`) → returns `{access_token, refresh_token, user, household_id}`.

JWT claims always include: `sub` (user_id), `email`, `is_superuser`, `household_id` (current active household), `jti`, `iat`, `exp`. Services that need household scoping read `household_id` directly from the token.

**Token lifecycle:**
- Access token: 30 min (configurable via `ACCESS_TOKEN_EXPIRE_MINUTES`)
- Refresh token: 14 days (configurable via `REFRESH_TOKEN_EXPIRE_DAYS`). Stored hashed in DB. **Rotated on `/auth/refresh`** — each refresh mints a new refresh token, chained to its parent by `family_id`/`parent_id`, with a short in-process grace window (`REFRESH_TOKEN_GRACE_SECONDS`, default 10s) so a benign double-submit re-gets the cached successor. Replaying an already-rotated token is rejected (401); it revokes the whole family **only** if `REFRESH_TOKEN_REVOKE_FAMILY_ON_REUSE=true` (default off — see Invariants). Clients MUST persist and use the newly returned refresh token.

### 3. App-to-app authentication (the most common path)

Service-A wants to call Service-B:
```
Service-A → Service-B: GET /something
  Headers: X-Jarvis-App-Id: jarvis-service-a
           X-Jarvis-App-Key: <key from .env>

Service-B → jarvis-auth: GET /internal/app-ping
  Headers: X-Jarvis-App-Id, X-Jarvis-App-Key forwarded
  → 200 if valid (returns app_id + name), 401 if not

Service-B → Service-A: 200 (or 401 on failure)
```

App credentials live in the `AppClient` table. They're created (and the raw key returned **once**) via `POST /admin/app-clients` or, more commonly, via `jarvis-config-service POST /v1/services/register` which orchestrates the whole bootstrap.

### 4. Node authentication

Nodes (Pi Zeros) authenticate via a node_id + node_key pair, but also have **per-service access grants** (`NodeServiceAccess`). A service calling `/internal/validate-node` passes a `service_id`; auth checks the node exists, is active, key matches, AND has access to that specific service. Today, **most node→service traffic flows through command-center**, so most nodes only need access to command-center (granted automatically when command-center calls `/internal/nodes/register`).

### 5. Household scoping

Every user belongs to ≥1 household. JWT carries the *active* household. Users can:
- Be in multiple households (`HouseholdMembership` table, unique on `(household_id, user_id)`)
- Switch active household via `POST /auth/switch-household` (re-issues JWT with the new `household_id`)
- Join via invite code (`POST /households/join` — invite codes are 16 chars, can have a `max_uses` and `expires_at`, can grant a non-default role)

Roles in a household: `MEMBER < POWER_USER < ADMIN`. The hierarchy is checked via `HouseholdRole.has_permission(user_role, required_role)`. Used by command-center to gate settings writes.

Multi-tenancy is **live and used** — the beta has external family members on a shared install, and they're scoped by household.

### 6. Settings (auth's own settings router)

Auth mounts the standard `jarvis-settings-client` router at `/settings/*`. Split auth model:
- **Reads** accept superuser JWT OR app-to-app credentials (so apps can display settings to users for context)
- **Writes** require superuser JWT only (so no service can mutate top-level settings without explicit user action)

This pattern is shared across every service that has settings. The settings table has multi-tenant scoping (`household_id`, `node_id`, `user_id`, all nullable) with cascade lookup `user > node > household > system default`.

---

## "How to..." recipes

### Add an app credential for a new service

Two paths:
- **From the installer / first-boot:** call config-service `POST /v1/services/register` — it'll create the credential here, fetch the raw key, and write it to the service's `.env` file. **This is the canonical path.**
- **Ad-hoc:** `POST /admin/app-clients` with `X-Jarvis-Admin-Token` header. The raw key is returned in the response **once** — store it. Subsequent reads only show metadata.

If a service is added to `known_services.py` in config-service, the admin UI will offer to register it; you don't need to touch this service.

### Add a new internal endpoint (service-callable)

Add to `jarvis_auth/app/api/internal.py`. Use `app_client: models.AppClient = Depends(require_app_client)` to gate on app credentials. The dependency lives in `jarvis_auth/app/api/dependencies/app_auth.py`.

### Add a new admin endpoint

Add to a new or existing file under `jarvis_auth/app/api/admin_*.py`. Mount the router with `dependencies=[Depends(require_admin_token)]` (from `jarvis_auth/app/api/dependencies/admin_auth.py`) — checks `X-Jarvis-Admin-Token` against `JARVIS_AUTH_ADMIN_TOKEN`.

Wire it up in `jarvis_auth/app/main.py:create_app()`.

### Add a new role check

`from jarvis_auth.app.db.models import HouseholdRole; HouseholdRole.has_permission(user_role, required_role)`. For command-center–style external authorization checks, use `POST /internal/validate-household-access` with `{household_id, user_id, required_role}`.

### Validate a JWT in another service

Don't round-trip. Each service reads `AUTH_SECRET_KEY` from env and decodes the JWT locally with `python-jose` (HS256). The `jarvis-settings-client` library bundles a `create_superuser_auth()` helper that's the canonical pattern.

---

## Invariants & gotchas

1. **`api/routes/*` is dead code.** A partial refactor introduced `api/routes/auth.py`, `api/routes/users.py`, and `api/routes/__init__.py` — but `main.py` does NOT wire these up. The active routers are in `api/auth.py`, `api/internal.py`, `api/admin_*.py`, `api/households.py`, `api/invites.py`, `api/superuser_views.py`. **Add new routes to `api/*.py`, not `api/routes/*.py`.**
2. **Env var is `AUTH_SECRET_KEY`, not `SECRET_KEY`.** Older docs (and the meta CLAUDE.md) say `SECRET_KEY` — this is wrong for jarvis-auth itself. All other services share the same secret under `SECRET_KEY` for JWT validation; here it's read as `AUTH_SECRET_KEY` via Pydantic alias.
3. **Refresh tokens ARE rotated** (since 2026-05, migration `a3b4c5d6e7f8`). `/auth/refresh` mints a new refresh token each call and marks the parent `rotated_at`; tokens are chained into a `family_id`. A `REFRESH_TOKEN_GRACE_SECONDS` (default 10s) in-process cache (`core/refresh_cache.py`) lets a benign double-submit of the just-rotated parent re-get the cached successor instead of failing. **Client code MUST adopt and persist the returned refresh token** — replaying a rotated one 401s. By default a stale replay does NOT revoke the family (a flaky-network mobile client replays far more often than tokens are stolen, and the in-process cache is lost on every restart → a restart-during-grace would otherwise nuke live sessions); set `REFRESH_TOKEN_REVOKE_FAMILY_ON_REUSE=true` to restore strict whole-family revocation. The grace cache is in-process only — auth must stay single-worker until it's backed by Redis/Postgres.
4. **JWT validation is local-only — revocation is eventually consistent.** Even after marking a user inactive or deleting them, their unexpired access token (≤30 min) still works in downstream services. If you need stricter revocation, shorten `ACCESS_TOKEN_EXPIRE_MINUTES`. There is **no** `/internal/validate-jwt` endpoint and adding one would invert the design.
5. **`JARVIS_AUTH_ADMIN_TOKEN` is special.** It's the master key for `/admin/*` endpoints. It's *only* used by trusted infrastructure (installer, config-service first-boot). Do not pass it from user-facing code or non-bootstrap contexts.
6. **`/auth/setup` is one-shot.** Once any superuser exists, it returns 409. There is no other supported path to create the first superuser. Treat as installer-only.
7. **Refresh tokens cascade-delete with their user.** `CASCADE` is on `RefreshToken.user_id` and `HouseholdMembership.user_id`. Deleting a user (admin action) revokes all their tokens automatically. The `Setting` table is NOT cascaded — settings scoped to `user_id` survive user deletion. Likely needs cleanup logic if you support user deletion.
8. **Password reset is admin-driven (no email).** A superuser issues a temp password via `POST /superuser/users/{id}/temp-password` (plaintext returned ONCE, never stored/logged; expires after `TEMP_PASSWORD_EXPIRE_HOURS`, default 24). This sets `users.must_change_password` and revokes every session. Login with the temp password succeeds but returns `must_change_password: true` — clients must gate on it and force `POST /auth/change-password`, which clears the flag, revokes all other sessions, and returns a fresh token pair the client MUST adopt. Server-side backstops: **both login AND refresh** reject inactive users (generic 401) and expired temp passwords (distinct 401), so a temp session can't outlive its expiry by rotating. Revocation and rotation serialize on a `SELECT ... FOR UPDATE` of the user row, revocation purges `core/refresh_cache` (via `services/token_revocation.py`), and the grace path never serves a successor for a revoked row.
9. **`get_db` in `api/auth.py` IS `deps.get_db` — never redefine it locally.** FastAPI caches dependencies by callable identity; a module-local duplicate gives `get_current_user` and the endpoint two different sessions, so mutations on `current_user` commit against the wrong session and are silently rolled back in prod while tests (which override both) stay green. Pinned by `test_auth_get_db_is_deps_get_db`.
10. **CORS allow-list:** browser clients are `jarvis-admin`, `jarvis-web`, and `jarvis-node-mobile`. Set `CORS_ALLOWED_ORIGINS` to a comma-separated list. Default is `http://localhost:5173` (admin dev server only).
11. **Settings table scoping is `(key, household_id, node_id, user_id)` unique.** A `key` can have one row per scope tuple. Cascade lookup: user → node → household → NULL (system default). Don't query directly — use `jarvis-settings-client`.

---

## Data model

```python
User                    # id, email (unique), username (unique), password_hash, is_active, is_superuser, must_change_password, temp_password_expires_at
RefreshToken            # user_id (CASCADE), token_hash, expires_at, revoked
AppClient               # app_id (unique), name, key_hash, is_active, last_rotated_at
Household               # id (UUID str), name
HouseholdMembership     # household_id, user_id, role (MEMBER|POWER_USER|ADMIN); UNIQUE(household, user)
HouseholdInvite         # household_id, code (16 chars), default_role, max_uses, use_count, expires_at, revoked
NodeRegistration        # node_id (unique), household_id, node_key_hash, name, is_active
NodeServiceAccess       # node_id (CASCADE), service_id; UNIQUE(node, service) — per-service access grants
Setting                 # key, value, value_type, category, env_fallback, multi-tenant scoping
```

All timestamps timezone-aware UTC. Models in `jarvis_auth/app/db/models.py`.

---

## API surface (active routes)

### User auth (`api/auth.py`, no auth required except where noted)
| Method | Path | Notes |
|---|---|---|
| POST | `/auth/register` | Auto-login. Optional `invite_code` body field or `X-Household-Id` header. Otherwise creates "My Home" household, user becomes admin. |
| POST | `/auth/login` | Email + password → tokens |
| POST | `/auth/refresh` | Refresh token in body → new access token **+ a newly rotated refresh token** (client must adopt it). 10s grace re-serves the successor on a benign double-submit. |
| POST | `/auth/change-password` | Bearer JWT. Verifies current password, revokes ALL sessions, returns a fresh token pair. Clears `must_change_password`. |
| POST | `/auth/logout` | Refresh token in body (no Bearer — works with an expired access token). Revokes that token's family; `all_devices: true` revokes every session. Always 204. |
| GET | `/auth/me` | Requires Bearer JWT |
| GET | `/auth/setup-status` | Public: `{needs_setup: bool}` |
| POST | `/auth/setup` | One-shot: creates first superuser. 409 if already done. |
| POST | `/auth/switch-household` | Requires Bearer JWT; re-issues JWT with the new `household_id` |

### Households (`api/households.py`, Bearer JWT)
| Method | Path | Notes |
|---|---|---|
| POST | `/households` | Create household; caller becomes admin |
| GET | `/households` | List current user's households |
| POST | `/households/{id}/leave` | Guards: only-household refused, last-admin refused, cascade cleans up |
| DELETE | `/households/{id}/members/{user_id}` | Admin-only |
| POST | `/households/join` | Body: invite code |

### Invites (`api/invites.py`, Bearer JWT)
| Method | Path | Notes |
|---|---|---|
| POST | `/invites` | Create invite. Body: household_id, default_role, max_uses?, expires_at? |
| (other CRUD per file) |  |  |

### Superuser views (`api/superuser_views.py`, Bearer JWT + `is_superuser` — the admin UI's surface)
| Method | Path | Notes |
|---|---|---|
| GET | `/superuser/households` | All households |
| GET | `/superuser/nodes` | All nodes |
| GET | `/superuser/users` | All users + household memberships + `must_change_password` |
| POST | `/superuser/users/{id}/temp-password` | Issue temp password (see Invariants #8). Body optional: `{temp_password?, expires_in_hours?}` |

### Admin: app clients (`api/admin_app_clients.py`, `X-Jarvis-Admin-Token`)
| Method | Path |
|---|---|
| POST | `/admin/app-clients` |
| GET | `/admin/app-clients` |
| POST | `/admin/app-clients/{app_id}/rotate` |
| POST | `/admin/app-clients/{app_id}/revoke` |

### Admin: nodes (`api/admin_nodes.py`, `X-Jarvis-Admin-Token`)
| Method | Path |
|---|---|
| POST | `/admin/nodes` |
| GET | `/admin/nodes` |
| GET | `/admin/nodes/{node_id}` |
| DELETE | `/admin/nodes/{node_id}` (deactivate) |
| POST | `/admin/nodes/{node_id}/rotate-key` |
| POST | `/admin/nodes/{node_id}/services` |
| DELETE | `/admin/nodes/{node_id}/services/{service_id}` |

### Admin: users (`api/admin_users.py`, `X-Jarvis-Admin-Token`)
| Method | Path |
|---|---|
| (CRUD per file) |  |

### Internal (`api/internal.py`, requires app credentials)
| Method | Path | Used by |
|---|---|---|
| GET | `/internal/app-ping` | Any service validating app creds |
| POST | `/internal/validate-node` | Any service accepting nodes (returns `household_id` + member IDs for voice ID) |
| POST | `/internal/nodes/register` | Services provisioning new nodes (auto-grants self) |
| POST | `/internal/nodes/{node_id}/services` | Service-side grant |
| DELETE | `/internal/nodes/{node_id}/services/{service_id}` | Service-side revoke |
| DELETE | `/internal/nodes/{node_id}` | Service-side deactivate |
| POST | `/internal/validate-household-access` | command-center settings RBAC |
| POST | `/internal/validate-node-household` | command-center settings RBAC |
| GET | `/internal/users/batch` | command-center speaker resolution (cap 100/req) |

### Settings (`/settings/*`, library mount)
- Reads: superuser JWT OR app credentials
- Writes: superuser JWT only

---

## Config surface

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `AUTH_SECRET_KEY` | yes | — | JWT signing key. **Must match `SECRET_KEY` in every other service** (they read the same secret under different alias). |
| `AUTH_ALGORITHM` | no | `HS256` | JWT algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | no | `30` | Access token lifetime |
| `REFRESH_TOKEN_EXPIRE_DAYS` | no | `14` | Refresh token lifetime |
| `TEMP_PASSWORD_EXPIRE_HOURS` | no | `24` | Admin-issued temp password lifetime |
| `DATABASE_URL` | yes | — | PostgreSQL connection string |
| `JARVIS_AUTH_ADMIN_TOKEN` | yes | — | Master admin token for `/admin/*` endpoints |
| `CORS_ALLOWED_ORIGINS` | no | `http://localhost:5173` | Comma-separated browser origins |
| `JARVIS_APP_ID` | no | `jarvis-auth` | App credential ID this service uses to send logs |
| `JARVIS_APP_KEY` | no | — | App credential key for logging |
| `JARVIS_LOG_CONSOLE_LEVEL` | no | `WARNING` | |
| `JARVIS_LOG_REMOTE_LEVEL` | no | `DEBUG` | |

---

## Architecture

```
jarvis_auth/app/
├── main.py                            # FastAPI factory + router wiring (THE source of truth for active routes)
├── core/
│   ├── settings.py                    # Pydantic Settings (pulls from env)
│   ├── security.py                    # JWT encode/decode, password hashing, token gen
│   └── logging.py                     # JarvisLogger setup
├── db/
│   ├── base.py                        # Declarative Base
│   ├── models.py                      # User, RefreshToken, AppClient, Household*, Node*, Setting
│   └── session.py                     # Engine, SessionLocal
├── schemas/                           # Pydantic request/response shapes
├── services/
│   ├── auth_service.py                # User register/login/refresh business logic
│   ├── user_service.py                # User lookups
│   └── settings_service.py            # Backing store for /settings router
├── api/
│   ├── auth.py                        # PUBLIC user auth (/auth/*)  ← actively mounted
│   ├── households.py                  # /households/*               ← actively mounted
│   ├── invites.py                     # /invites/*                  ← actively mounted
│   ├── internal.py                    # /internal/*                 ← actively mounted (app-creds gated)
│   ├── admin_app_clients.py           # /admin/app-clients/*        ← actively mounted (admin-token gated)
│   ├── admin_nodes.py                 # /admin/nodes/*              ← actively mounted
│   ├── admin_users.py                 # /admin/users/*              ← actively mounted
│   ├── deps.py                        # get_current_user, require_superuser, get_db
│   ├── dependencies/
│   │   ├── admin_auth.py              # require_admin_token (X-Jarvis-Admin-Token)
│   │   └── app_auth.py                # require_app_client (X-Jarvis-App-Id/Key)
│   └── routes/                        # ⚠️ DEAD CODE — see Invariants #1
└── alembic/                           # Migrations
```

---

## Testing

- **Unit tests only.** Integration tests are on the roadmap but blocked on environment setup. Same constraint as config-service.
- Test DB is SQLite via `pytest` fixtures; schema created via `Base.metadata.create_all`, not Alembic.
- Auth dependencies are overridden via FastAPI `dependency_overrides`.
- When adding a new route: write a TestClient test asserting status, response shape, and (if applicable) the role-check rejection path.

Run: `poetry run pytest`.

---

## Failure modes

| Failure | Behavior |
|---|---|
| Postgres down | Service won't start; cascade failure |
| jarvis-logs down | Logs go to console; no other impact |
| `AUTH_SECRET_KEY` mismatch between auth and any service | That service rejects all JWTs as invalid → 401 |
| `JARVIS_AUTH_ADMIN_TOKEN` unset/wrong | All `/admin/*` requests 401 |
| Refresh token expired | `/auth/refresh` returns 401; client must re-login |
| Node deactivated mid-session | Next call to `/internal/validate-node` returns `valid=false` → service rejects |
| Superuser deactivated | Their JWT keeps working until expiry (≤30 min); refresh is rejected, so the session ends there. For instant lockout, also issue a temp-password reset (revokes all refresh tokens). |

---

## Out of scope / explicitly not here

- **Email-based password reset.** No email service in the stack; resets are admin-driven temp passwords (Invariants #8).
- **SSO / OAuth / OIDC.** Local user/password only.
- **Instant access-token revocation.** Logout/change-password/reset revoke refresh tokens only; an unexpired access token (≤30 min default) keeps working downstream because JWT validation is local (Invariants #4).
- **Per-resource RBAC.** Only household-level roles; finer-grained authorization lives in each service.
- **Audit log.** No `audit_events` table. Logs go to jarvis-logs as JSON events; query there if you need an audit trail.

---
> Source: [alexberardi/jarvis-auth](https://github.com/alexberardi/jarvis-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
