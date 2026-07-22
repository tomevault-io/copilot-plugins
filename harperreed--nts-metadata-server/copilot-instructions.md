## nts-metadata-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands

```bash
# Build the application
go build -o main .

# Run locally (starts on :8080)
go run main.go

# Run tests
go test -v ./...

# Run tests with coverage
go test -v ./... -coverprofile=coverage.out -covermode=atomic

# Run tests with race detector
go test -race -v ./...

# Run integration tests (requires external API)
INTEGRATION_TESTS=1 go test -v ./...

# Docker build and run
docker build -t nts-metadata .
docker run -p 8080:8080 nts-metadata

# Deploy to Fly.io
fly deploy
```

## Architecture Overview

This is a modular Go application organized into packages:

1. **HTTP Server** (`main.go`): Uses Gorilla Mux router with two main routes:
   - `/api/live/{channel}` - Returns JSON metadata for NTS live channels (1 or 2)
   - `/api/mixtapes/{alias}` - Returns JSON metadata for NTS infinite mixtapes
   - `/` - Serves static HTML documentation page

2. **Cache Layer** (`cache/`): Generic in-memory cache with per-key TTL support
   - Thread-safe with mutex protection
   - Supports different TTLs per key
   - Request de-duplication to prevent thundering herd

3. **NTS Client** (`nts/`): HTTP client wrapper for NTS API v2
   - `FetchLive()` - Fetches `/live` endpoint
   - `FetchMixtapes()` - Fetches `/mixtapes` endpoint
   - Handles timeouts, size limits, error responses

4. **Handlers** (`handlers/`): HTTP request handlers
   - `LiveHandler` - Extracts specific channel from full live response (5s TTL)
   - `MixtapeHandler` - Extracts specific mixtape from full mixtapes response (30s TTL)
   - Both generate ETags and support If-None-Match

5. **ETag Support**: Generates ETags using SHA-256 hashes of response content to enable client-side caching.

## Key Implementation Details

- **Channel Validation**: Live channels must be "1" or "2"
- **Mixtape Lookup**: Mixtapes are looked up by alias from the full API response
- **Error Handling**: All errors are properly handled and returned as JSON with appropriate HTTP status codes
- **CORS Headers**: Enabled for browser compatibility
- **Testing**: Comprehensive test suite with mocked HTTP clients for unit tests
- **Deployment**: Auto-deploys to Fly.io via GitHub Actions on push to main branch

---
> Source: [harperreed/nts-metadata-server](https://github.com/harperreed/nts-metadata-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
