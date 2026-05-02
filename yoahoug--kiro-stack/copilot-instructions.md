## kiro-stack

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kiro Stack integrates two upstream projects to provide OpenAI/Anthropic-compatible API access to Kiro (Amazon Q Developer) accounts:

- **kiro-go** (Go): Web admin panel, multi-account pool management, token refresh
- **kiro-gateway** (Python/FastAPI): Stable proxy layer with retry logic and dual-endpoint fallback

## Architecture

```
Client (Claude Code/Cursor/etc) → kiro-go:8088 → kiro-gateway:8000 → Kiro API
```

**Request Flow:**
1. Client sends OpenAI/Anthropic-compatible request to kiro-go
2. kiro-go selects account from pool (weighted random selection)
3. If `KIRO_GATEWAY_BASE` is set, kiro-go forwards to kiro-gateway with account credentials in headers
4. kiro-gateway handles token refresh, retries, and dual-endpoint fallback
5. Response streams back through the chain

**Key Integration Point:** `kiro-go/proxy/handler.go:380-395` (`proxyToGatewayWithAccount`) injects account credentials as HTTP headers (`X-Kiro-Refresh-Token`, `X-Kiro-Region`, etc.) when forwarding to gateway.

## Common Commands

### Development

```bash
# Start both services with Docker Compose
docker compose up -d

# View logs
docker compose logs -f kiro-go
docker compose logs -f kiro-gateway

# Rebuild after code changes
docker compose up -d --build

# Stop services
docker compose down
```

### kiro-go (Go service)

```bash
cd kiro-go

# Build
go build -o kiro-go .

# Run locally
./kiro-go

# Run with custom config path
CONFIG_PATH=/path/to/config.json ./kiro-go

# Format code
go fmt ./...
```

### kiro-gateway (Python service)

```bash
cd kiro-gateway

# Install dependencies
pip install -r requirements.txt

# Run locally
python main.py

# Run with custom port
python main.py --port 9000

# Run tests
pytest

# Run specific test file
pytest tests/unit/test_auth.py

# Run with verbose output
pytest -v
```

## Configuration

### Environment Variables

**Root .env file (simplified setup):**
- `ADMIN_PASSWORD` - Web admin panel password (required)
- `INTERNAL_API_KEY` - Internal communication key between kiro-go and kiro-gateway (required)
- `VPN_PROXY_URL` - HTTP/SOCKS5 proxy for restricted networks (optional)
- `DEBUG_MODE` - Logging mode: `off`, `errors`, `all` (optional)

**How it works:**
- All configuration is in the root `.env` file
- `docker-compose.yml` reads from `.env` and distributes variables to both services
- kiro-go manages all account credentials
- When forwarding requests, kiro-go injects credentials as HTTP headers (`X-Kiro-*`)
- kiro-gateway receives credentials from headers, no need for separate .env configuration

### Configuration Files

**kiro-go:** `data/config.json` stores accounts, settings, and statistics. Thread-safe access via `config/config.go` with RWMutex.

**kiro-gateway:** No configuration file needed when used with kiro-go. Receives all credentials via HTTP headers.

## Code Structure

### kiro-go (Go)

**Entry point:** `main.go` - Initializes config, account pool, and HTTP handler

**Core modules:**
- `config/config.go` - Configuration management with JSON persistence. Account struct includes auth credentials, subscription info, usage tracking, and runtime statistics
- `pool/account.go` - Account pool with weighted random selection. Maintains enabled accounts in memory
- `proxy/handler.go` - HTTP routing and request handling. Contains gateway proxy logic (lines 160-176, 380-395)
- `proxy/kiro.go` - Direct Kiro API client (used when gateway is not configured)
- `proxy/translator.go` - Request/response translation between OpenAI/Anthropic and Kiro formats
- `auth/` - Authentication methods: AWS Builder ID, IAM SSO, OIDC token refresh

**Gateway Integration:** When `KIRO_GATEWAY_BASE` is set, `NewHandler()` creates a reverse proxy (`handler.go:163-176`). The `proxyToGatewayWithAccount()` function (`handler.go:380-395`) injects account credentials as headers before forwarding.

### kiro-gateway (Python)

**Entry point:** `main.py` - FastAPI app with lifespan management

**Core modules:**
- `kiro/config.py` - Environment variable configuration
- `kiro/auth.py` - KiroAuthManager handles token refresh and authentication
- `kiro/auth_pool.py` - Builds authentication pool from credentials
- `kiro/routes_openai.py` - OpenAI-compatible `/v1/chat/completions` endpoint
- `kiro/routes_anthropic.py` - Anthropic-compatible `/v1/messages` endpoint
- `kiro/model_resolver.py` - Model name normalization and resolution
- `kiro/cache.py` - Model info caching

**Request Headers:** Gateway expects account credentials in `X-Kiro-*` headers (set by kiro-go). If not present, falls back to environment variables.

## Important Patterns

### Account Pool Selection

`pool/account.go` uses weighted random selection. Each account has a `Weight` field (default: 100). Higher weight = higher selection probability. Only enabled accounts are included in the pool.

### Token Refresh

Both services handle token refresh:
- **kiro-go:** Background refresh every 30 minutes (`handler.go:184-203`). Checks if token expires within 5 minutes and refreshes proactively
- **kiro-gateway:** Refreshes on-demand when token is expired or about to expire

### Retry Logic

**kiro-go:** When using gateway, implements failover with up to 4 attempts (`handler.go:397+`). Tries different accounts on failure.

**kiro-gateway:** Implements dual-endpoint fallback (CodeWhisperer vs Amazon Q endpoints) with configurable retry logic.

### Thinking Mode

Both services support extended thinking mode. Triggered by model name suffix (default: `-thinking`). Configuration in `config.json` (`ThinkingSuffix`, `OpenAIThinkingFormat`, `ClaudeThinkingFormat`).

## Testing

**kiro-gateway only:** Uses pytest. Test structure:
- `tests/unit/` - Unit tests for individual modules
- `tests/integration/` - Integration tests
- `tests/conftest.py` - Shared fixtures

**kiro-go:** No test suite currently exists.

## Utilities

**scripts/sync_tokens.py** - Syncs refresh tokens from kiro-go config to kiro-gateway .env file. Reads enabled accounts from `data/config.json` and writes `KIRO_REFRESH_TOKENS` to `.env` as comma-separated list. Useful for keeping gateway's token pool in sync with go service accounts.

## Deployment Notes

- Both services bind to `0.0.0.0` by default for Docker compatibility
- kiro-go stores persistent data in `data/config.json` - mount this volume in production
- Gateway is stateless - no persistent storage required
- Services communicate via internal Docker network when using docker-compose

---
> Source: [Yoahoug/kiro-stack](https://github.com/Yoahoug/kiro-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
