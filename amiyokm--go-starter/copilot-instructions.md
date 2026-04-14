## go-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Motamo backend — a Go API for a mobile app's authentication system. Built with Gin (HTTP), GORM (ORM), PostgreSQL, and Redis. Implements registration, login, email verification, password reset, refresh token rotation, logout, user profile endpoints, and OAuth (Google/Facebook). Auth uses JWT access tokens + rotating refresh tokens stored in the database.

## Commands

```bash
# Development (hot-reload via Air)
air

# Build
go build -o ./bin/main ./cmd/api

# Run directly
go run ./cmd/api

# Database migrations
make db/migrations/create name=<name>   # create new migration
make db/migrations/up                   # apply migrations
make db/migrations/down                 # rollback migrations

# Tests
go test ./...                           # run all tests
go test ./service/auth/...              # run tests for a package

# Dependencies
go mod tidy
```

## Architecture

Strict layered architecture: **handler → service → repository**. Every layer depends only on interfaces, never on concrete types from other layers. Dependency injection is manual, wired in `cmd/api/main.go`.

```
handler/     → HTTP handlers (Gin), input validation, JSON responses
  ├── handler.go       # Composition root for all handlers
  ├── auth/            # Auth domain (register, login, verify, reset, logout, OAuth)
  │   ├── handler.go
  │   └── port.go      # Service interfaces (Auth + OAuth) consumed by handler
  └── users/           # Users domain (get me, get/update profile)
      ├── handler.go
      └── port.go      # Service interface consumed by handler

service/     → Business logic, depends on repository interfaces
  ├── service.go       # Composition root for all services
  ├── auth/            # Auth business logic
  │   ├── service.go
  │   └── port.go      # Repository + peer-service interfaces
  ├── oauth/           # Google/Facebook OAuth flow
  │   ├── service.go
  │   └── port.go      # Repository + peer-service interfaces
  ├── jwt/             # JWT (HS256) access token generation & validation
  │   └── service.go
  ├── refreshToken/    # Refresh token lifecycle (create, rotate, revoke)
  │   ├── service.go
  │   └── port.go      # Repository interface
  ├── tokens/          # Generic token generation & SHA-256 hashing
  │   └── service.go
  ├── email/           # Resend API integration
  │   └── service.go
  └── users/           # User profile business logic
      ├── service.go
      └── port.go      # Repository interface + TxManager interface

repository/  → GORM data access, generic CRUD via gorm.Interface[T]
  ├── repository.go    # Composition root for all repos + TxManager
  ├── auth/
  ├── userProfile/
  ├── emailVerficationToken/
  ├── passwordResetToken/
  ├── refreshToken/
  └── oauthAccount/

models/      → GORM entity structs (embed gorm.Model)
middleware/  → Auth middleware (AuthRequired with JWT), rate limiting, context helpers
config/      → EnvConfig, DbConfig, RedisConfig structs
utils/       → Shared utilities (errors, response, time, env parsing)
infra/       → DB connection, Redis client
```

All routes are under `/api/v1/` and registered in `cmd/api/routes.go`.

## How the Layers Wire Together

```
main.go
  │
  ├── repository.New(db)                                    # DB → all repos
  │     returns: Repository {
  │       Auth, UserProfile, EmailVerificationToken,
  │       PasswordResetToken, OAuthAccount, RefreshToken
  │     }
  │
  ├── repository.NewTxManager(db)                           # DB → transaction manager
  │
  ├── cache.New(cfg.Redis)                                  # Redis client for OAuth state
  │
  ├── service.NewService(repo, txManager, envConfig, redis) # Repos + txManager + config + Redis → all services
  │     wires: JWTService from env (JWT_SECRET, ACCESS_TOKEN_EXPIRATION)
  │     wires: RefreshTokenService gets Repository{RefreshToken} + env
  │     wires: AuthService gets Repository{Auth,EmailVerificationToken,UserProfile,PasswordResetToken}
  │     wires:   + TxManager + Service{Email, JWT, RefreshToken} + env
  │     wires: OAuthService gets Repository{OAuthAccount,Auth,UserProfile}
  │     wires:   + TxManager + Service{JWT, RefreshToken} + env + Redis
  │     wires: UserService gets Repository{UserProfile} + TxManager
  │     returns: Service {
  │       Email, Auth, Token, Users, OAuth, JWT, RefreshToken
  │     }
  │
  ├── middleware.NewMiddleware()                             # Rate limiters only (no args)
  │
  ├── svc.Auth implements middleware.Authenticator           # ValidateAccessToken + GetUserByID
  │
  └── handler.NewHandler(svc)                               # Services → all handlers
        returns: Handler {
          Auth, Users
        }
```

## Routes

```
Public (with AuthRateLimiter):
  POST   /api/v1/auth/register
  PATCH  /api/v1/auth/verify-email
  POST   /api/v1/auth/login
  POST   /api/v1/auth/forgot-password
  POST   /api/v1/auth/reset-password
  POST   /api/v1/auth/refresh
  GET    /api/v1/auth/google
  GET    /api/v1/auth/google/callback
  GET    /api/v1/auth/facebook
  GET    /api/v1/auth/facebook/callback

Protected (with AuthRequired):
  POST   /api/v1/auth/logout
  POST   /api/v1/auth/logout-all
  POST   /api/v1/auth/link/google
  POST   /api/v1/auth/link/facebook
  GET    /api/v1/users/me
  GET    /api/v1/users/me/profile
  PATCH  /api/v1/users/me/profile
```

## Key Patterns & Rules

### Adding a New Feature (strict order)

1. **Repository first** — add DB methods to `repository/<domain>/repository.go`. If new table, create the package and register in `repository/repository.go`.
2. **Service second** — add business methods to `service/<domain>/service.go`. Declare new repo/service interfaces in `service/<domain>/port.go`.
3. **Handler third** — add HTTP methods to `handler/<domain>/handler.go`. Add service interface methods to `handler/<domain>/port.go`. Parse request → call service → return response. **No business logic in handlers.**
4. **Route last** — register in `cmd/api/routes.go`.
5. **Wire in `main.go`** — if new structs were created, wire them in the composition chain.

### Port Pattern (Interfaces)

Each domain defines its dependencies as **interfaces in `port.go`** — not concrete types:

- `handler/<domain>/port.go` — defines `Service` struct with service interfaces the handler consumes
- `service/<domain>/port.go` — defines `Repository` struct with repo interfaces, and `Service` struct with peer-service interfaces

**Never import a concrete service or repo type across layer boundaries.** The handler imports only `handler/auth/port.go` interfaces, not `service/auth/`.

### Generic GORM Repos

Repositories use `gorm.G[T]` and `gorm.Interface[T]` for type-safe CRUD:

```go
type MyRepository struct {
    db       *gorm.DB
    myModel  gorm.Interface[models.MyModel]
}

func NewMyRepository(db *gorm.DB) *MyRepository {
    model := gorm.G[models.MyModel](db)
    return &MyRepository{db: db, myModel: model}
}
```

**CRITICAL: `Updates` (plural) takes a value, not a pointer.** Use `r.myModel.Where(...).Updates(ctx, *ptr)` not `Updates(ctx, ptr)`. `Update` (singular) takes a field name and value: `r.myModel.Where(...).Update(ctx, "field", value)`.

### Error Handling

- **Sentinel errors** live in `utils/errors.go` (e.g., `ErrUserAlreadyExists`, `ErrTokenNotFound`, `ErrRefreshTokenInvalid`, `ErrRefreshTokenReuse`, `ErrOAuthAccountAlreadyLinked`)
- **Response helpers** in `utils/errors.go`: `RespondBadRequest`, `RespondUnauthorized`, `RespondForbidden`, `RespondNotFound`, `RespondConflict`, `RespondValidationError`, `RespondInternalError`, `RespondOK`, `RespondCreated`
- Handlers use sentinel errors + switch on `errors.Is()` to map to HTTP status codes
- Services wrap errors with `fmt.Errorf("context: %w", err)` for debugging
- Email failures are logged with `log.Printf("WARN: ...")` but don't fail the request (user/token already persisted)

### Password Reset & Email Links

- Email links point to **`FRONTEND_URL`**, not `API_URL` — browsers can't send POST/PATCH from email links. The frontend extracts the token and makes the API call.
- `ForgotPassword` **always returns nil** — never leaks email existence. Log warnings but return success.
- `ResetPassword` revokes all refresh tokens after password change — forces re-login everywhere.

### Naming Conventions

- Service **structs** and service **interfaces** in the same package must have different names (e.g., `UserService` struct vs `UserService` interface causes "redeclared" compile error). Use `UserService` for the struct and `UserService` interface in `port.go`, or differentiate them.
- Use `*models.UserProfile` for partial updates — **never `map[string]interface{}`**. GORM's `Updates` with a struct skips zero-value fields automatically.
- Route paths use kebab-case: `/logout-all`, not `/logoutAll`.
- DB column names are snake_case: `password_hash`, `user_id`, `display_name`.

### Auth Middleware

- `middleware.AuthRequired(authenticator)` validates Bearer JWT token via `Authenticator` interface, attaches `user` to Gin context
- `Authenticator` interface: `ValidateAccessToken(tokenString string) (uint, error)` + `GetUserByID(ctx, id) (*models.User, error)` — implemented by `AuthService`
- Retrieve user via `middleware.GetUser(c)` — returns `*models.User` or `nil`
- All protected endpoints must be under the `protected` route group with `AuthRequired()` middleware
- Nil checks on context values are defensive — middleware guarantees they're set, but check anyway

### JWT & Refresh Token Architecture

- **Access tokens**: Short-lived JWT (HS256), default 15 min. Generated/validated by `service/jwt/`. Contains `userID` claim.
- **Refresh tokens**: Long-lived random tokens (256-bit), SHA-256 hashed before storage. Stored in `refresh_tokens` table with `token_hash`, `parent_token_hash`, `expires_at`, `revoked_at`.
- **Token rotation**: On `Refresh`, the old token is revoked and a new one is created with `parent_token_hash` pointing to the old token's hash. This creates a chain for reuse detection.
- **Reuse detection**: If a revoked token is used, the entire chain (all tokens descended from it) is revoked — security measure against token theft.
- `RefreshTokenService` owns all refresh token lifecycle: `CreateRefreshToken`, `RotateRefreshToken`, `RevokeRefreshToken`, `RevokeAllRefreshTokens`
- `AuthService` delegates to `RefreshTokenService` via the `Service.RefreshToken` interface — the handler never calls `RefreshTokenService` directly

### Auth Flows

- **Login**: Returns `(accessToken, refreshToken, user, error)`. Creates a new refresh token.
- **VerifyEmail**: Returns `(accessToken, refreshToken, error)`. Auto-logs the user in after verification by creating tokens.
- **Refresh**: Rotates refresh token, returns `(accessToken, newRefreshToken, error)`.
- **Logout**: Revokes the provided refresh token.
- **LogoutAll**: Revokes all refresh tokens for the user.
- **ResetPassword**: Updates password + marks token used (in transaction), then revokes all refresh tokens (outside transaction).
- **OAuth (Google/Facebook)**: Redirect → callback exchange → upsert user → return `(accessToken, refreshToken, user)`.
- **OAuth Linking** (protected): Links a Google/Facebook account to an existing authenticated user.

### Transaction Management

- `repository.TxManager` wraps `gorm.DB.Transaction()` — call `repository.NewTxManager(db)` in `main.go`
- Each domain that needs transactions defines a `TxManager` interface + adapter in its `port.go`:
  - `service/auth/port.go`: `AuthTxManagerAdapter` — maps `repository.Repository` to `auth.Repository`
  - `service/oauth/port.go`: `OAuthTxManagerAdapter` — maps `repository.Repository` to `oauth.Repository`
  - `service/users/port.go`: `UsersTxManagerAdapter` — maps `repository.Repository` to `users.Repository`
- **All multi-step mutations in AuthService use `s.tx.WithTx()`**:
  - `RegisterUser`: creates user + profile + verification token atomically
  - `VerifyEmail`: marks email verified + marks token used atomically
  - `ResetPassword`: updates password + marks token used atomically (refresh token revocation happens after)
- Inside `WithTx` callbacks, use the `txRepo` parameter (transaction-scoped repository), NOT `s.repo`
- If any step fails inside `WithTx`, the entire transaction rolls back — no partial state
- Email sends happen AFTER the transaction commits (outside the `WithTx` callback)

### Config & Environment

- `godotenv` loads `.env` on startup
- Typed access via `utils/env.go`: `GetString(key, default)`, `GetInt(key, default)`, `GetBool(key, default)`
- All env vars loaded in `cmd/api/main.go` into `config.EnvConfig`
- New env vars must be added to: `config/config.go`, `cmd/api/main.go`, `.env`, `.env.example`

Key env vars by domain:
- **Auth**: `TOKEN_HASH_SALT`, `JWT_SECRET`, `ACCESS_TOKEN_EXPIRATION` (default 15m), `REFRESH_TOKEN_EXPIRATION` (default 720h/30d)
- **OAuth**: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REDIRECT_URL`, `GOOGLE_USERINFO_URL`, `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET`, `FACEBOOK_REDIRECT_URL`, `FACEBOOK_USERINFO_URL`
- **Email**: `RESEND_API_KEY`, `FROM_EMAIL`, `FRONTEND_URL`, `EMAIL_VERIFICATION_TOKEN_EXPIRATION`, `PASSWORD_RESET_TOKEN_EXPIRATION`
- **DB/Redis**: `DB_ADDR`, `DB_MAX_CONN_OPEN`, `DB_MAX_IDLE_CONN`, `DB_MAX_IDLE_TIME`, `REDIS_ADDR`

### Email Service

- Uses Resend API with templates (`service/email/service.go`)
- Template IDs: `welcome-email` (verification), `password-reset`
- Both methods use `frontendUrl` from `FRONTEND_URL` env var
- Both methods implement retry logic with `MAX_EMAIL_SEND_RETRIES` (3 attempts) — intermediate failures logged as WARN, final failure returned
- Email failure during registration is logged but doesn't fail the request (user/token already persisted)

### Rate Limiting

- `middleware/rateLimit.go` — IP-based rate limiting using `golang.org/x/time/rate`
- **Global**: 10 req/sec, burst 20, TTL 60s — applied to all routes
- **Auth**: 1 req/12sec, burst 3, TTL 60s — applied to auth public routes only

## Infrastructure

- **Docker**: `docker-compose.yml` provides PostgreSQL 17.9 (port 5432) and Redis 6.2 (port 6379)
- **Container name**: `motamo-be-postgres-db`, database: `motamao_db`
- **Email**: Resend API for transactional emails (`service/email/`)
- **Config structs**: Defined in `config/` — `Config` aggregates `EnvConfig`, `DbConfig`, `RedisConfig`

## Design Spec

The full planned API is documented in `docs/superpowers/specs/2026-04-02-mobile-auth-api-design.md`. The implementation plan with step-by-step instructions is in `docs/superpowers/plans/2026-04-02-mobile-auth-api.md`. Use both as sources of truth.

## Common Gotchas

1. **`Updates` vs `Update`**: `Updates(ctx, value)` takes a struct/value, `Update(ctx, "field", value)` takes a field name. `Updates` with a struct skips zero-value fields (partial update). `Updates` on `gorm.Interface[T]` takes `T` (value), not `*T` — dereference pointers: `Updates(ctx, *profile)`.
2. **No `map[string]interface{}`**: Use typed structs for partial updates. Build a struct with only the fields to set; GORM skips zero values.
3. **Service struct vs interface naming**: Don't name a struct and interface the same thing in one package — causes "redeclared" error.
4. **Email links use `FRONTEND_URL`**: Not `API_URL`. Browsers can't POST from email links.
5. **ForgotPassword always returns 200**: Never leaks whether an email exists. Log warnings, return nil.
6. **Protected routes go under `protected` group**: Not the public `authGroup`. Must have `AuthRequired()` middleware.
7. **Column names are snake_case**: `password_hash`, `user_id`, `display_name` — match the DB migration, not Go struct field names.
8. **`GetByHash` returns `nil, nil` on not found**: Not an error. The service layer checks for `nil` to determine "not found".
9. **Handler never imports `service/<domain>`**: The handler uses interfaces from `handler/<domain>/port.go` only.
10. **Commit refactors separately**: Don't mix repository refactors with feature commits.
11. **Refresh token revocation after ResetPassword**: Happens outside the transaction, not inside — if revocation fails, the password is still changed but old tokens may remain valid.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AmiyoKm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AmiyoKm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
