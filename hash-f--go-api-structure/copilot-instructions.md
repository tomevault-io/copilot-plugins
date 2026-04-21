## go-api-structure

> - Dont reference full filenames like this cci:1://file:///Users/h/Developer/go/go-api-structure/internal/logger/logger.go:7:0-31:1


Follow:

- Dont reference full filenames like this cci:1://file:///Users/h/Developer/go/go-api-structure/internal/logger/logger.go:7:0-31:1
- Dont dont add info about the commands you ran.

Example commit message:
feat(api): Implement /users/me endpoint and migrate to pgx
Implemented the GET /api/v1/users/me endpoint:

- Added UserHandler with GetMe method in internal/api/handler_user.go.
- Wired the handler and route in internal/server/server.go, protected by JWT auth.

Refactored database layer to use pgx/v5:

- Created internal/database/database.go with NewPgxPool for pgxpool.Pool.
- Updated cmd/api/main.go to use pgxpool.Pool and pass it to store.NewStore.
- Resolved associated type mismatches and lint errors.

Enhanced configuration:

- Made JWT expiry duration configurable via JWT_EXPIRY_MINUTES in internal/config/config.go.
- Updated internal/server/server.go to use this configured value.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hash-f) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
