## go-reddit-storage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go library providing database persistence for Reddit data fetched via `go-reddit-api-wrapper`. It supports both PostgreSQL and SQLite backends with identical interfaces.

## Key Commands

### Testing
```bash
# Run all tests
go test -v ./...

# Run specific package tests
go test -v ./sqlite
go test -v ./postgres

# Run PostgreSQL tests (requires DATABASE_URL)
export TEST_POSTGRES_URL="postgres://user:pass@localhost/reddit_test?sslmode=disable"
go test -v ./postgres

# Start test PostgreSQL with Docker
docker run -d --name test-postgres \
  -e POSTGRES_PASSWORD=test \
  -e POSTGRES_DB=reddit_test \
  -p 5432:5432 \
  postgres:15
```

### Building
```bash
# Build CLI tool
go build -o reddit-archiver ./cmd/reddit-archiver

# Install CLI globally
go install github.com/jamesprial/go-reddit-storage/cmd/reddit-archiver@latest
```

### Running Examples
```bash
# Set required environment variables
export REDDIT_CLIENT_ID="your-client-id"
export REDDIT_CLIENT_SECRET="your-client-secret"

# Run examples
go run examples/basic/main.go
go run examples/continuous/main.go
go run examples/backfill/main.go

# Run CLI tool
./reddit-archiver -subreddit golang -limit 100 -comments
```

## Architecture

### Storage Interface Pattern
The codebase uses an interface-based design ([storage.go](storage.go)) that allows multiple backend implementations. Both PostgreSQL and SQLite implement the same `Storage` interface, providing:
- Post/comment CRUD operations
- Subreddit metadata storage
- Full-text search (PostgreSQL only)
- Query options for filtering and pagination

### Archiver Pattern
The `Archiver` type ([archiver.go](archiver.go)) combines a Reddit API client with a storage backend to provide high-level operations:
- `ArchiveSubreddit` - Fetch and store posts from a subreddit
- `ArchivePost` - Fetch and store a single post with comments
- `ContinuousArchive` - Monitor and archive new content continuously
- `BackfillSubreddit` - Archive historical posts with pagination
- `UpdateScores` - Refresh scores for recently archived posts

### Database Schema & Migrations
- Schema files are embedded in the binary using `go:embed` ([schema/schema.go](schema/schema.go))
- Migrations are stored in [schema/migrations/postgres/](schema/migrations/postgres/) and [schema/migrations/sqlite/](schema/migrations/sqlite/)
- Migration runner automatically tracks applied versions in `schema_version` table
- Migrations run transactionally with automatic rollback on failure
- Add new migrations as `NNN_description.sql` where NNN is the version number

### Idempotent Operations
All save operations use UPSERT patterns:
- PostgreSQL: `INSERT ... ON CONFLICT DO UPDATE`
- SQLite: `INSERT OR REPLACE`
This ensures re-archiving the same content is safe and updates existing records.

### Batch Operations
- `SavePosts` and `SaveComments` use transactions and prepared statements
- Batch operations are significantly faster than individual saves
- All batch operations are atomic (all succeed or all fail)

### Comment Threading
- Comments store `depth` field and `parent_id` references
- Recursive CTEs are used to query full comment trees (see [postgres/comments.go](postgres/comments.go))
- Comment depth is calculated during archiving based on Reddit's structure

### Backend-Specific Implementation
Each backend (postgres/, sqlite/) is organized into:
- `<backend>.go` - Connection, migrations, close, subreddit operations
- `posts.go` - Post-specific CRUD operations
- `comments.go` - Comment-specific CRUD operations
- `<backend>_test.go` - Backend-specific tests

### Error Handling
The codebase uses `StorageError` type ([storage.go](storage.go:56)) for all storage-related errors. This wraps underlying errors with operation context.

## Common Development Patterns

### Adding New Storage Methods
1. Add method to `Storage` interface in [storage.go](storage.go)
2. Implement in [postgres/postgres.go](postgres/postgres.go) (or appropriate file)
3. Implement in [sqlite/sqlite.go](sqlite/sqlite.go) (or appropriate file)
4. Add tests to both `postgres_test.go` and `sqlite_test.go`

### Adding New Migrations
1. Create `00X_description.sql` in both `schema/migrations/postgres/` and `schema/migrations/sqlite/`
2. Increment version number from last migration
3. Migrations run automatically on next `RunMigrations()` call
4. Test migrations on both backends

### Testing with Real Databases
- PostgreSQL tests require `TEST_POSTGRES_URL` environment variable
- SQLite tests use temporary files (no setup required)
- Use [internal/testutil/fixtures.go](internal/testutil/fixtures.go) for test data generation

## Important Notes

- Always use context for cancellation support in long-running operations
- Store raw JSON in `raw_json` columns for future schema evolution
- PostgreSQL connection strings require `sslmode` parameter
- SQLite uses WAL mode for better concurrency (enabled automatically)
- The CLI tool reads credentials from environment variables (never hardcode)
- Comment depth is stored denormalized for query performance
- Always commit to git after finishing anything

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JamesPrial) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
