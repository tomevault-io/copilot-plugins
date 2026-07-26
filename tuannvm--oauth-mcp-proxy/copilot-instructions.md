## oauth-mcp-proxy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**oauth-mcp-proxy** is an OAuth 2.1 authentication library for Go MCP servers. It provides server-side OAuth integration with minimal code (3-line integration via `WithOAuth()`), supporting multiple providers (HMAC, Okta, Google, Azure AD).

**Version**: v1.0.0 (Supports both `mark3labs/mcp-go` and official `modelcontextprotocol/go-sdk`)

## Build Commands

```bash
# Run tests
make test

# Run tests with verbose output
make test-verbose

# Run tests with coverage report (generates coverage.html)
make test-coverage

# Run linters (same as CI - checks go.mod tidy + golangci-lint)
make lint

# Format code
make fmt

# Clean build artifacts and caches
make clean

# Install/download dependencies
make install

# Check for security vulnerabilities
make vuln
```

## Architecture

### Package Structure (v1.0.0)

```
oauth-mcp-proxy/
├── [core package - SDK-agnostic]
│   ├── oauth.go         - Server type, NewServer, ValidateTokenCached
│   ├── config.go        - Configuration validation and provider setup
│   ├── cache.go         - Token cache with 5-minute TTL
│   ├── context.go       - Context utilities (WithOAuthToken, GetUserFromContext, etc.)
│   ├── handlers.go      - OAuth HTTP endpoints (/.well-known/*, /oauth/*)
│   ├── middleware.go    - CreateHTTPContextFunc for token extraction
│   ├── logger.go        - Logger interface
│   ├── metadata.go      - OAuth metadata structures
│   └── provider/        - Token validators (HMAC, OIDC)
│
├── mark3labs/          - Adapter for mark3labs/mcp-go SDK
│   ├── oauth.go        - WithOAuth → ServerOption
│   └── middleware.go   - Middleware for mark3labs types
│
└── mcp/                - Adapter for official modelcontextprotocol/go-sdk
    └── oauth.go        - WithOAuth → http.Handler
```

### Core Components

**Core Package** (SDK-agnostic):
1. **oauth.go** - `Server` type, `NewServer()`, `ValidateTokenCached()` (used by adapters)
2. **config.go** - Configuration validation and provider setup
3. **cache.go** - Token caching logic (`TokenCache`, `CachedToken`)
4. **context.go** - Context utilities (`WithOAuthToken`, `GetOAuthToken`, `WithUser`, `GetUserFromContext`)
5. **handlers.go** - OAuth HTTP endpoints
6. **provider/provider.go** - Token validators (HMACValidator, OIDCValidator)

**Adapters** (SDK-specific):
- **mark3labs/** - Middleware adapter for `mark3labs/mcp-go`
- **mcp/** - HTTP handler wrapper for official SDK

### Key Design Patterns

- **OpenTelemetry Pattern**: Core logic is SDK-agnostic; adapters provide SDK-specific integration
- **Instance-scoped**: Each `Server` instance has its own token cache and validator (no globals)
- **Provider abstraction**: `TokenValidator` interface supports multiple OAuth providers
- **Caching strategy**: Tokens cached for 5 minutes using SHA-256 hash as key
- **Context propagation**: OAuth token extracted from HTTP header → stored in context → validated → user added to context

### Integration Flow

**mark3labs SDK:**
```text
1. HTTP request with "Authorization: Bearer <token>" header
2. CreateHTTPContextFunc() extracts token → adds to context via WithOAuthToken()
3. mark3labs middleware validates token:
   - Calls Server.ValidateTokenCached() (checks cache first)
   - If not cached, validates via provider (HMAC or OIDC)
   - Caches result (5-minute TTL)
4. Adds authenticated User to context via WithUser()
5. Tool handler accesses user via GetUserFromContext(ctx)
```

**Official SDK:**
```text
1. HTTP request with "Authorization: Bearer <token>" header
2. mcp adapter's HTTP handler intercepts request
3. Validates token via Server.ValidateTokenCached():
   - Checks cache first (5-minute TTL)
   - If not cached, validates via provider
   - Caches result
4. Adds token and user to context (WithOAuthToken, WithUser)
5. Passes request to official SDK's StreamableHTTPHandler
6. Tool handler accesses user via GetUserFromContext(ctx)
```

### Provider System

- **HMAC**: Validates JWT tokens with shared secret (testing/dev)
- **OIDC**: Validates tokens via JWKS/OIDC discovery (Okta/Google/Azure)
- All validation happens in `provider/provider.go`
- Validators implement `TokenValidator` interface

## Testing

The codebase has extensive test coverage across multiple scenarios:

- **api_test.go** - Core API functionality tests
- **integration_test.go** - End-to-end integration tests
- **security_test.go** - Security validation tests
- **attack_scenarios_test.go** - Security attack scenario tests
- **middleware_compatibility_test.go** - Middleware compatibility tests
- **provider/provider_test.go** - Token validator tests

Run single test:

```bash
go test -v -run TestName ./...
```

### Test Patterns

Tests use **table-driven subtests** with `t.Run()`:

```go
tests := []struct {
    name string
    // test fields
}{...}
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        // test body
    })
}
```

Mock validators implement `TokenValidator` interface. Use `httptest.NewRecorder()` for HTTP handler tests.

## Configuration

### ConfigBuilder Pattern (Recommended)

Use `ConfigBuilder` for production code instead of direct `Config` structs:

```go
cfg, _ := oauth.NewConfigBuilder().
    WithProvider("okta").
    WithIssuer("https://company.okta.com").
    WithAudience("api://my-server").
    WithHost(host).WithPort(port).
    Build()
```

`Build()` validates config and auto-constructs `ServerURL` if not set.

### Context Timeouts

- **OIDC validation**: 10 seconds
- **Provider initialization**: 30 seconds

## Security Requirements

1. **Redirect URI validation**: All URIs must be in explicit allowlist
2. **State parameter HMAC**: OAuth states are HMAC-signed to prevent CSRF
3. **Audience validation**: Both HMAC and OIDC validators explicitly check `aud` claim
4. **No raw token logging**: Only log `fmt.Sprintf("%x", sha256.Sum256([]byte(token)))[:16]`
5. **TLS in production**: Always warn if `useTLS=false` in `LogStartup()`

## Important Notes

1. **User Context**: Always use `GetUserFromContext(ctx)` in tool handlers to access authenticated user
2. **Token Caching**: Tokens cached for 5 minutes - design for this TTL in testing. Cache uses `sync.RWMutex` with background cleanup via `deleteExpiredToken()` goroutine
3. **Logging**: Config.Logger is optional. If nil, uses default logger (log.Printf with level prefixes)
4. **Modes**: Library supports "native" (token validation only) and "proxy" (OAuth flow proxy) modes. Auto-detected based on `ClientID` presence
5. **Adapter Pattern**: `WithOAuth()` is in adapter packages (`mark3labs.WithOAuth()` or `mcp.WithOAuth()`) for SDK-specific integration

## Common Gotchas

1. **SDK Imports**: Adapter code (`mark3labs/`, `mcp/`) can import SDKs. **Core package cannot** - keep it SDK-agnostic
2. **Context Propagation**: Always extract user via `GetUserFromContext(ctx)` in tool handlers
3. **Cache Expiry**: Background cleanup runs in goroutine to avoid lock contention
4. **Mode Detection**: Config auto-detects "native" vs "proxy" based on `ClientID` presence
5. **Logger Fallback**: If `cfg.Logger == nil`, uses `defaultLogger{}` with `log.Printf`

## File Naming Conventions

- Core logic: `oauth.go`, `config.go`, `cache.go`, `context.go`, `handlers.go`, `middleware.go`
- Tests: `*_test.go` (e.g., `security_test.go`, `integration_test.go`)
- Adapters: `mark3labs/oauth.go`, `mcp/oauth.go` (not `*_adapter.go`)
- Provider: `provider/provider.go` (single file, multiple validators)

## Using the Library

### With mark3labs/mcp-go
```go
import (
    oauth "github.com/tuannvm/oauth-mcp-proxy"
    "github.com/tuannvm/oauth-mcp-proxy/mark3labs"
)

oauthServer, oauthOption, _ := mark3labs.WithOAuth(mux, &oauth.Config{...})
mcpServer := server.NewMCPServer("name", "1.0.0", oauthOption)

streamableServer := server.NewStreamableHTTPServer(mcpServer, ...)
mux.HandleFunc("/mcp", oauthServer.WrapMCPEndpoint(streamableServer))
```

**Note**: `WrapMCPEndpoint()` provides automatic 401 handling with proper WWW-Authenticate headers when Bearer token is missing. It also passes through OPTIONS requests (CORS) and non-Bearer auth schemes.

### With Official SDK
```go
import (
    oauth "github.com/tuannvm/oauth-mcp-proxy"
    mcpoauth "github.com/tuannvm/oauth-mcp-proxy/mcp"
)

mcpServer := mcp.NewServer(&mcp.Implementation{...}, nil)
_, handler, _ := mcpoauth.WithOAuth(mux, &oauth.Config{...}, mcpServer)
http.ListenAndServe(":8080", handler) // 401 handling automatic
```

**Note**: Official SDK adapter includes automatic 401 handling in the returned handler.

## Extending the Library

### Adding a New OAuth Provider

1. Add validator to `provider/provider.go` implementing `TokenValidator` interface
2. Update `createValidator()` switch in `config.go`
3. Add provider documentation in `docs/providers/`

### Adding a New SDK Adapter

1. Create `<sdk>/oauth.go` with `WithOAuth()` function
2. Follow pattern: create `oauth.Server`, register handlers, return SDK-specific middleware/option
3. Never import MCP SDKs in core package

### Adding New Endpoints

1. Add handler method to `OAuth2Handler` in `handlers.go`
2. Register in `RegisterHandlers()` in `oauth.go`

## Documentation References

- `examples/README.md` - Complete setup guide with Okta configuration
- `examples/mark3labs/` and `examples/official/` - Working examples (simple + advanced)
- `docs/providers/*.md` - Provider-specific setup (OKTA.md, GOOGLE.md, AZURE.md, HMAC.md)
- `docs/CONFIGURATION.md` - All configuration options
- `docs/SECURITY.md` - Production best practices
- `docs/TROUBLESHOOTING.md` - Common issues and solutions

---
> Source: [tuannvm/oauth-mcp-proxy](https://github.com/tuannvm/oauth-mcp-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
