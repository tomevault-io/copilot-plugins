## mavt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MAVT (Mobile App Version Tracker) is a Go application that monitors Apple App Store applications for version updates. It provides both CLI and daemon modes for tracking app versions over time.

**Key Features:**
- Track multiple iOS apps by bundle ID
- Detect and record version changes with release notes
- Web UI with App Store search for easy app discovery
- Run as a daemon for continuous monitoring
- Apprise integration for notifications (Discord, Slack, Telegram, email, 80+ services)
- REST API for programmatic access
- Docker containerization with GHCR images
- No database required - JSON file-based storage

## Development Commands

### Go Module Management
```bash
# Initialize Go module (if not already done)
go mod init github.com/username/mavt

# Download dependencies
go mod download

# Tidy dependencies
go mod tidy

# Verify dependencies
go mod verify
```

### Building
```bash
# Build the project
go build ./...

# Build specific package
go build ./path/to/package

# Build with output binary
go build -o mavt ./cmd/mavt
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests with race detector (important for concurrent code)
go test -race ./...

# Run tests with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...

# View coverage in browser
go tool cover -html=coverage.out

# Run specific test
go test -run TestName ./path/to/package

# Run tests matching a pattern
go test -run TestName.* ./...
```

**Note:** The codebase currently has no test files. When adding tests, place them alongside the code they test with `_test.go` suffix.

### Code Quality
```bash
# Format code
go fmt ./...

# Run go vet for static analysis
go vet ./...

# Run golangci-lint (if installed)
golangci-lint run
```

### Running
```bash
# Show version information
go run ./cmd/mavt -version

# Add an app to track
go run ./cmd/mavt -add com.apple.mobilesafari

# List all tracked apps
go run ./cmd/mavt -list

# Check for updates immediately
go run ./cmd/mavt -check

# Run as daemon (continuous monitoring)
go run ./cmd/mavt -daemon

# Show version history for an app
go run ./cmd/mavt -updates com.apple.mobilesafari

# Show recent updates (e.g., last 24 hours)
go run ./cmd/mavt -recent 24h

# Export data to a ZIP backup
go run ./cmd/mavt -export backup.zip

# Import data from a ZIP backup
go run ./cmd/mavt -import backup.zip

# Build and run the binary
go build -o mavt ./cmd/mavt
./mavt -version
./mavt -add com.apple.Music
```

### Docker Commands
```bash
# Build Docker image
docker build -t mavt:latest .

# Create named volume before first run (required by docker-compose)
docker volume create mavt-data

# Set proper permissions on volume (run as user 1000:1000)
docker run --rm -v mavt-data:/data alpine sh -c "chown -R 1000:1000 /data && ls -la /data"

# Run with docker-compose (daemon mode)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop container
docker-compose down

# Pull pre-built image from GHCR
docker pull ghcr.io/thomas/mavt:latest

# Run one-time check
docker run --rm -v $(pwd)/data:/app/data \
  -e MAVT_APPS=com.apple.mobilesafari \
  mavt:latest -check

# Interactive shell in container
docker-compose exec mavt sh
```

## Project Structure

```
mavt/
├── cmd/mavt/              # Main application entry point
│   └── main.go            # CLI commands, flag parsing, and daemon orchestration
├── internal/              # Private application code (Go convention: not importable)
│   ├── appstore/          # iTunes/App Store API client
│   │   └── client.go      # HTTP client for App Store lookups and search
│   ├── backup/            # Data backup and restore
│   │   └── backup.go      # ZIP export/import with path traversal and zip bomb protection
│   ├── config/            # Configuration management
│   │   └── config.go      # Environment variable loader (MAVT_* prefix)
│   ├── notifier/          # Apprise notification integration
│   │   └── notifier.go    # HTTP client for sending notifications
│   ├── server/            # HTTP server and web UI
│   │   └── server.go      # Web interface and REST API endpoints
│   ├── storage/           # Data persistence layer
│   │   └── storage.go     # JSON file-based storage with RWMutex
│   ├── tracker/           # Core tracking logic
│   │   └── tracker.go     # Version monitoring, comparison, and update detection
│   └── version/           # Version information
│       └── version.go     # Version constants (updated for releases)
├── pkg/models/            # Public data models
│   └── app.go             # AppInfo and VersionUpdate structs
├── data/                  # Runtime data directory (gitignored)
│   ├── apps/              # Current app information (one JSON per bundle ID)
│   └── updates/           # Version update history (array of updates per bundle ID)
├── Dockerfile             # Multi-stage Docker build
├── docker-compose.yml     # Docker Compose configuration (uses named volume)
└── .env.example           # Example environment variables
```

## Web Interface

When running in daemon mode, MAVT provides a web interface on the configured port (default: 8080).

### Accessing the Web UI
- Default URL: `http://localhost:8080` (or your configured `MAVT_SERVER_PORT`)
- In Docker: Exposed on the mapped port (e.g., `http://localhost:7738`)

### Web Features
- **Search & Add Apps**: Search the App Store and add apps to tracking with one click
- **View Tracked Apps**: See all apps being monitored with version info
- **Recent Updates**: View version changes from the last 7 days
- **Auto-refresh**: Page updates every 30 seconds

## Architecture

### Data Flow
1. **Configuration** ([internal/config/config.go](internal/config/config.go)) loads environment variables on startup
2. **App Store Client** ([internal/appstore/client.go](internal/appstore/client.go)) queries iTunes Search API by bundle ID, track ID, or search term
3. **Tracker** ([internal/tracker/tracker.go](internal/tracker/tracker.go)) orchestrates version checks and compares with stored data
4. **Storage** ([internal/storage/storage.go](internal/storage/storage.go)) persists app info and version updates as JSON files
5. **Notifier** ([internal/notifier/notifier.go](internal/notifier/notifier.go)) sends update notifications via Apprise when changes detected
6. **HTTP Server** ([internal/server/server.go](internal/server/server.go)) provides web UI and REST API
7. **Main** ([cmd/mavt/main.go](cmd/mavt/main.go)) handles CLI flags, daemon mode loop, and signal handling

### Key Components

**Tracker (internal/tracker/tracker.go):**
- Core business logic for version comparison
- Calls App Store API for current version data
- Compares with stored version to detect changes
- Preserves `FirstDiscovered` timestamp across updates
- Updates `LastChecked` timestamp on every check
- Includes `sanitizeForLog()` to prevent log injection attacks
- Triggers notifications when updates are found

**Storage (internal/storage/storage.go):**
- File-based JSON storage in `data/` directory
- `data/apps/{bundleID}.json` - Current app information (single object)
- `data/updates/{bundleID}.json` - Version history (array of updates)
- Thread-safe with `sync.RWMutex` for concurrent access
- Directory creation handled automatically

**App Store API Integration (internal/appstore/client.go):**
- Uses iTunes Search API (`https://itunes.apple.com/lookup` and `https://itunes.apple.com/search`)
- No authentication required
- Supports country/region selection (ISO 3166-1 alpha-2 codes)
- Returns app metadata including version, release notes, release date
- Rate limiting: Be respectful, no official limit documented

**Notifier (internal/notifier/notifier.go):**
- Optional Apprise integration for multi-service notifications
- Supports direct service URLs (Discord, Slack, Telegram, etc.)
- Batches multiple updates into single notification
- Truncates long release notes to 500 chars
- 10 second HTTP timeout for notification requests

**HTTP Server (internal/server/server.go):**
- Embedded web UI with HTML/CSS/JS
- REST API for app management and search
- Runs concurrently with daemon loop in goroutine
- Dynamic sync interval display based on config

**Configuration (internal/config/config.go):**
- Environment-based using `MAVT_*` prefixed variables
- Parses durations (e.g., "1h", "30m") for check intervals
- Supports comma-separated app lists for batch tracking
- See [.env.example](.env.example) for all options

**Version Management (internal/version/version.go):**
- Version constant updated manually for releases
- `BuildDate` and `GitCommit` set via ldflags during build
- Displayed with `-version` flag

### API Endpoints

The HTTP server provides the following REST API endpoints:

**GET /**
- Web UI dashboard

**GET /api/apps**
- Returns all tracked apps as JSON
- Example: `curl http://localhost:8080/api/apps`

**GET /api/updates?since=<duration>**
- Returns recent version updates
- Duration format: `1h`, `24h`, `7d`, `168h`
- Example: `curl http://localhost:8080/api/updates?since=24h`

**GET /api/history?bundle_id=<bundle_id>**
- Returns version history for a specific app
- Query parameter: `bundle_id` (required)
- Example: `curl "http://localhost:8080/api/history?bundle_id=com.burbn.instagram"`

**GET /api/search?q=<term>&limit=<number>**
- Search App Store by name
- `q`: Search term (required)
- `limit`: Max results (optional, default 10, max 50)
- Example: `curl "http://localhost:8080/api/search?q=instagram&limit=5"`

**POST /api/track**
- Add an app to tracking
- Body: `{"bundle_id": "com.example.app"}`
- Example: `curl -X POST -H "Content-Type: application/json" -d '{"bundle_id":"com.burbn.instagram"}' http://localhost:8080/api/track`

**GET /api/last-update**
- Returns the timestamp of the most recent version update detected

**GET /api/export**
- Downloads a ZIP backup of all tracked app data

**POST /api/import**
- Imports a ZIP backup (multipart form upload)
- Validates against zip bombs (50MB total, 5MB per file, 2000 files max) and path traversal

**GET /api/health**
- Health check endpoint
- Returns: `{"status":"healthy","tracked_apps":N,"timestamp":"..."}`

### Daemon Mode Architecture

When run with `-daemon` flag ([cmd/mavt/main.go](cmd/mavt/main.go:189)):
1. Sets up context with cancellation for graceful shutdown
2. Registers signal handlers for SIGINT/SIGTERM
3. Starts HTTP server in separate goroutine
4. Performs initial check immediately
5. Starts ticker loop for periodic checks (interval from config)
6. Both server and ticker run concurrently until shutdown signal

**Concurrency Considerations:**
- HTTP server runs in goroutine, independent of check loop
- Storage uses RWMutex for thread-safe file access
- Tracker checks all apps serially to avoid API rate limits
- Notification failures are logged but don't halt the operation

### Adding New Features

When extending functionality:
- **App Store API interactions**: Add methods to `internal/appstore/client.go`
- **Business logic**: Extend `internal/tracker/tracker.go` methods
- **HTTP/API endpoints**: Add handlers to `internal/server/server.go`
- **Data models**: Public models in `pkg/models/`, internal ones in respective `internal/*/` packages
- **CLI commands**: Add flags and handlers to `cmd/mavt/main.go`
- **Configuration**: Add new env vars to `internal/config/config.go` and document in `.env.example`
- **Notifications**: Extend `internal/notifier/notifier.go` for new notification types
- **Backup/restore**: Extend `internal/backup/backup.go` for data import/export logic

### Version Releases

When creating a new release:
1. Update version constant in [internal/version/version.go](internal/version/version.go)
2. Build with ldflags to inject build metadata:
   ```bash
   go build -ldflags "-X github.com/thomas/mavt/internal/version.BuildDate=$(date -u +%Y-%m-%dT%H:%M:%SZ) -X github.com/thomas/mavt/internal/version.GitCommit=$(git rev-parse HEAD)" -o mavt ./cmd/mavt
   ```
3. Tag release in git: `git tag v1.x.x`
4. Push tag: `git push origin v1.x.x`

## Environment Variables

See [.env.example](.env.example) for complete list. Key variables:
- `MAVT_APPS` - Comma-separated bundle IDs to track
- `MAVT_CHECK_INTERVAL` - How often to check (e.g., "1h", "30m", "4h")
- `MAVT_COUNTRY` - App Store region as ISO 3166-1 alpha-2 code (default: `AU`, e.g., "US", "GB")
- `MAVT_DATA_DIR` - Where to store tracking data (default: `./data`)
- `MAVT_LOG_LEVEL` - Logging verbosity (debug, info, warn, error)
- `MAVT_SERVER_HOST` - HTTP server bind address (default: `0.0.0.0`)
- `MAVT_SERVER_PORT` - HTTP server port (default: `8080`)
- `MAVT_APPRISE_URL` - Apprise notification URL (optional, enables notifications if set)
- `MAVT_ENVIRONMENT` - Environment name (default: `production`, options: `staging`, `production`)

## Deployment & Monitoring

### Railway Deployments

MAVT is deployed to Railway with separate staging and production environments:

- **Staging**: https://mavt-staging.up.railway.app/
- **Production**: https://mavt-production.up.railway.app/

Both environments build from source on Railway (not using GHCR images).

**Environment-Specific Features:**
- Staging displays a commit badge in the footer (e.g., `v1.1.9 abc1234`) when `MAVT_ENVIRONMENT=staging` is set
- Production does not show the commit badge for cleaner UI
- Commit badge requires `BuildDate` and `GitCommit` to be set via Docker build args (only available when using GHCR images)

### GitHub Actions Docker Builds

The [.github/workflows/docker-publish.yml](.github/workflows/docker-publish.yml) workflow automatically builds and publishes Docker images to GHCR:

**Triggers:**
- Push to `main` branch → builds `latest` tag
- Push to `staging` branch → builds `staging` tag
- Push tags matching `v*.*.*` → builds semver tags (e.g., `1.1.9`, `1.1`, `1`)

**Build Arguments:**
- `GIT_COMMIT` - Injected from `${{ github.sha }}`
- `BUILD_DATE` - Injected from `${{ github.event.head_commit.timestamp }}`

**Platforms:**
- `linux/amd64`
- `linux/arm64`

**Images Published to:** `ghcr.io/hoiber/mavt`

### Health Monitoring with Gatus

MAVT provides health check endpoints suitable for monitoring with [Gatus](https://github.com/TwiN/gatus) or similar tools.

**Recommended Gatus Configuration:**

```yaml
endpoints:
  # Staging Environment
  - name: MAVT Staging - Health
    url: "https://mavt-staging.up.railway.app/api/health"
    interval: 60s
    conditions:
      - "[STATUS] == 200"
      - "[BODY].status == healthy"
      - "[RESPONSE_TIME] < 3000"
    alerts:
      - type: discord  # or slack, telegram, etc.
        description: "MAVT Staging health check failed"

  - name: MAVT Staging - Web UI
    url: "https://mavt-staging.up.railway.app/"
    interval: 120s
    conditions:
      - "[STATUS] == 200"
      - "[BODY] == pat(*MAVT*)"
      - "[RESPONSE_TIME] < 3000"

  # Production Environment
  - name: MAVT Production - Health
    url: "https://mavt-production.up.railway.app/api/health"
    interval: 60s
    conditions:
      - "[STATUS] == 200"
      - "[BODY].status == healthy"
      - "[RESPONSE_TIME] < 3000"
    alerts:
      - type: discord
        description: "MAVT Production health check failed"

  - name: MAVT Production - Web UI
    url: "https://mavt-production.up.railway.app/"
    interval: 120s
    conditions:
      - "[STATUS] == 200"
      - "[BODY] == pat(*MAVT*)"
      - "[RESPONSE_TIME] < 3000"

  - name: MAVT Production - Apps API
    url: "https://mavt-production.up.railway.app/api/apps"
    interval: 300s
    conditions:
      - "[STATUS] == 200"
      - "[RESPONSE_TIME] < 3000"

  - name: MAVT Production - Updates API
    url: "https://mavt-production.up.railway.app/api/updates?since=24h"
    interval: 300s
    conditions:
      - "[STATUS] == 200"
      - "[RESPONSE_TIME] < 3000"
```

**Monitored Endpoints:**
- `/api/health` - Health check with JSON response validation
- `/` - Web UI availability and content verification
- `/api/apps` - Apps API availability
- `/api/updates?since=24h` - Updates API availability

---
> Source: [hoiber/mavt](https://github.com/hoiber/mavt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
