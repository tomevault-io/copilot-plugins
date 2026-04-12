## keycloak-cocoon-ai

> Shared Keycloak authentication library for Cocoon AI Python backends. Published as `keycloak-cocoon-ai`.

## Project Overview

Shared Keycloak authentication library for Cocoon AI Python backends. Published as `keycloak-cocoon-ai`.

Consumed by portal, billie, and spyder. Each project has its own Keycloak client but shares the same realm (`cai-portal`) and server (`https://auth.cocoon-ai.com`).

Companion to the TypeScript frontend package: [Cocoon-AI/keycloak-auth](https://github.com/Cocoon-AI/keycloak-auth).

## Architecture

Two auth patterns, both supported:

1. **Session-based OAuth** (target for new HTMX/Alpine.js apps) — `oauth_session.py`. Server does the OAuth dance, browser gets session cookies. Spyder uses this.
2. **SPA token auth** (for existing React apps) — `dependencies.py`. Browser uses keycloak-js, sends Bearer tokens. Portal and billie use this.

## Directory Structure

```
pyproject.toml                  # hatchling build, optional extras: [fastapi], [oauth], [dev]
src/keycloak_cocoon_ai/
  __init__.py                     # Re-exports core: config, models, exceptions, validators
  config.py                       # KeycloakConfig + AuthConfig (pydantic models with cocoon-ai defaults)
  models.py                       # KeycloakUser — base model returned by all auth deps
  exceptions.py                   # AuthError → NotAuthenticatedError, TokenExpiredError, InsufficientRoleError
  jwt_validator.py                # validate_keycloak_token() — RS256 JWKS validation via PyJWKClient
  alb_auth.py                     # authenticate_from_alb_headers() — decode ALB OIDC JWT (pre-verified)
  local_auth.py                   # create_local_token() / validate_local_token() — HS256 dev tokens
  fastapi/
    __init__.py
    dependencies.py               # create_auth_dependencies(AuthConfig) → {current_user, require_role, optional_auth}
    oauth_session.py              # create_session_auth(OAuthSessionConfig) → {current_user, create_oauth_routes, ...}
  testing/
    __init__.py
    mocks.py                      # mock_user(), mock_admin() factories
tests/
  conftest.py                     # RSA key fixtures, token factories
  test_models.py
  test_jwt_validator.py           # Mocks PyJWKClient
  test_alb_auth.py
  test_local_auth.py
  test_dependencies.py            # FastAPI TestClient integration tests
```

## Key Design Decisions

- **PyJWT over python-keycloak** — Lighter, no transitive deps. `python-keycloak` pulls in `requests`, `python-jose`, etc.
- **Factory pattern, not globals** — `create_auth_dependencies(config)` returns configured deps. No module-level singletons. Each app gets its own instance.
- **Custom exceptions converted at boundary** — Core code raises `AuthError` subclasses. The FastAPI layer catches them and re-raises as `HTTPException` with correct status codes.
- **azp not aud** — Keycloak access tokens use `azp` (authorized party) instead of `aud`. JWT validation skips audience check and verifies `azp` matches the client_id.
- **Token via query param** — SSE/EventSource can't set headers, so `?token=` is supported as fallback.

## Dependencies

- **Core**: `PyJWT`, `cryptography`, `pydantic`, `httpx`
- **[fastapi]**: `fastapi`
- **[oauth]**: `authlib`, `itsdangerous`
- **[dev]**: all of the above + `pytest`, `pytest-asyncio`

## Common Commands

```bash
source .venv/bin/activate       # venv already exists
pytest tests/ -v                # run all 33 tests
pytest tests/test_dependencies.py -v   # just FastAPI integration tests
```

## Auth Chain (SPA dependencies)

The `current_user` dependency in `dependencies.py` tries three modes in order:

1. **ALB**: Checks `x-amzn-oidc-identity` + `x-amzn-oidc-data` headers. ALB has already verified the token, so we just decode the JWT payload. Skipped if `alb_auth_enabled=False`.
2. **Keycloak RS256**: Extracts Bearer token from header (or `?token=` query param). Validates signature via JWKS, checks issuer and azp. Clock skew tolerance via `leeway` (default 60s).
3. **Local HS256**: Only tried if `dev_mode=True` and Keycloak validation failed. Uses `dev_secret` for symmetric validation.

If expired at any stage → 401. If all modes fail → 401. Wrong role → 403.

## KeycloakUser Fields

```python
sub: str              # Keycloak subject ID (always present)
email: str            # Default ""
name: str | None
picture: str | None
realm_roles: list[str]          # From realm_access.roles
auth_source: str                # "keycloak" | "alb" | "local" | "session" | "mock"
```

Private `_client_roles: dict[str, list[str]]` accessed via `get_client_roles(client_id)` and `has_client_role(client_id, role)`.

## Testing Approach

- JWT validator tests mock `PyJWKClient` and use real RSA keys generated in fixtures
- ALB auth tests construct fake JWT payloads (header.payload.sig format, no real signing)
- FastAPI dependency tests use `TestClient` with real HS256 tokens against the full dep chain
- `mock_user()` / `mock_admin()` are for consumer app tests via `app.dependency_overrides`

## Important Patterns

### Exception handling in FastAPI deps

Core modules raise `AuthError` subclasses. The `dependencies.py` wraps the core `_authenticate()` function and catches `AuthError`, converting to `HTTPException`:

```python
async def current_user(...):
    try:
        return await _authenticate(request, credentials)
    except AuthError as e:
        _raise_http(e)  # → HTTPException(status_code=e.status_code)
```

This keeps the core modules framework-agnostic while giving FastAPI proper HTTP responses.

### JWKS client caching

`jwt_validator.py` caches `PyJWKClient` instances by JWKS URL in a module-level dict. This means the first token validation for a given Keycloak config fetches the JWKS keys, and subsequent validations reuse the cached client. Tests clear this cache in `conftest.py`.

## Cocoon AI Defaults

- Server URL: `https://auth.cocoon-ai.com`
- Realm: `cai-portal`
- Client ID: `cai-portal-backend`

Override per-app by passing a custom `KeycloakConfig(client_id="billie")`.

## Consumers

| Project | Pattern |
|---------|---------|
| **Portal** | SPA token auth (`create_auth_dependencies`) |
| **Billie** | SPA token auth (`create_auth_dependencies`) |
| **Spyder** | Session OAuth (`create_session_auth`) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cocoon-AI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Cocoon-AI)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
