## good

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server that provides access to Go documentation from pkg.go.dev. The server exposes two main tools:
- `searchPackages`: Search for Go packages by query
- `getPackageInfo`: Get detailed documentation for a specific package

## Architecture

```
cmd/godoc-mcp-server/     # Main MCP server application
├── main.go              # Entry point, sets up MCP server with tools
├── server.go            # Note: Previously contained custom parameter helpers, now simplified
├── search.go            # searchPackages tool implementation  
└── packageInfo.go       # getPackageInfo tool implementation

pkg/godoc/               # Core documentation extraction logic
├── client.go            # HTTP client for pkg.go.dev
├── search.go            # Package search functionality
├── package.go           # Package documentation parsing
└── cache.go             # Local caching with freecache
```

The server supports both stdio and HTTP transports using the `github.com/mark3labs/mcp-go` framework. It scrapes pkg.go.dev using HTTP requests and HTML parsing with goquery.

## Key Components

### MCP Tools Architecture
- Each tool is defined with schema and handler function using `mcp.NewTool()`
- Tools use built-in MCP methods for parameter extraction:
  - `request.RequireString(key)` for required string parameters
  - `request.GetBool(key, default)` for optional boolean parameters
  - `request.GetString(key, default)` for optional string parameters
- All responses are JSON-marshaled strings returned via `mcp.NewToolResultText()`

### Documentation Extraction  
- `pkg/godoc/package.go` contains the core HTML parsing logic
- Extracts: overview, constants, variables, functions, types (with methods), and subpackages
- Uses CSS selectors to target specific elements on pkg.go.dev pages
- Supports optional URL extraction for definition links

### Caching Layer
- Local cache using `github.com/coocood/freecache` 
- Cache keys are function name + parameters (e.g., "getPkg" + package name)
- Reduces repeated HTTP requests to pkg.go.dev

## Development Commands

### Build
```bash
go build -o godoc-mcp-server ./cmd/godoc-mcp-server
```

### Cross-platform Build (matches CI)
```bash
# Linux ARM64
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -ldflags="-s -w" -o godoc-mcp-server-linux-arm64 ./cmd/godoc-mcp-server

# macOS ARM64  
GOOS=darwin GOARCH=arm64 CGO_ENABLED=0 go build -ldflags="-s -w" -o godoc-mcp-server-darwin-arm64 ./cmd/godoc-mcp-server

# Windows AMD64
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -ldflags="-s -w" -o godoc-mcp-server-windows-amd64.exe ./cmd/godoc-mcp-server
```

### Running the Server

#### Stdio Mode (Default)
```bash
./godoc-mcp-server
```

#### HTTP Mode  
```bash
# Run on default localhost:8080
./godoc-mcp-server -http

# Custom host and port
./godoc-mcp-server -http -host 0.0.0.0 -port 9000
```

#### Available HTTP Endpoints
- `GET /mcp/health` - Health check
- `POST /mcp/initialize` - Initialize MCP session
- `POST /mcp/tools/list` - List available tools
- `POST /mcp/tools/call` - Call a specific tool

### Install from Source
```bash
go install github.com/yikakia/godoc-mcp-server/cmd/godoc-mcp-server@latest
```

### Testing
No test files are present in the codebase. If adding tests:
```bash
go test ./...
go test -v ./pkg/godoc/
```

### Code Quality
```bash
go fmt ./...
go vet ./...
go mod tidy
```

## Important Implementation Details

### Parameter Handling (Updated for mcp-go v0.32.0+)
- Use built-in MCP parameter methods instead of custom helpers:
  - `request.RequireString(key)` - validates presence and type for required strings
  - `request.RequireBool(key)` - validates presence and type for required booleans  
  - `request.GetString(key, defaultValue)` - optional strings with defaults
  - `request.GetBool(key, defaultValue)` - optional booleans with defaults
- Methods handle type conversion and validation automatically
- Eliminates need for custom `requiredParam[T]()` and `OptionalParamOK[T]()` functions

### HTML Parsing Patterns
- Constants/Variables: `.Documentation-constants/.Documentation-variables` sections
- Functions: `.Documentation-functions .Documentation-function` blocks  
- Types: `.Documentation-types .Documentation-type` with nested `.Documentation-typeMethod`
- Subpackages: `table[data-test-id='UnitDirectories-table']` with special directory handling

### Error Handling
- Uses `github.com/pkg/errors` for error wrapping
- `go.uber.org/multierr` for collecting multiple errors
- MCP tool errors returned as `mcp.NewToolResultError()`

### Cache Implementation
Cache key format: `functionName + parameters`
- Search: `"search" + query`  
- Package info: `"getPkg" + packageName`

## Tool Usage Tips

When implementing new features:
1. The MCP tool descriptions are critical - they guide LLM usage patterns
2. Package name handling for subpackages requires concatenation (parent + "/" + subpackage)
3. URL extraction is optional and controlled by `needURL` parameter
4. All data structures are designed for JSON serialization to MCP clients

## Breaking Changes & Compatibility

### mcp-go v0.32.0+ Migration
The project has been updated to use the latest mcp-go version. Key changes:
- **Parameter Access**: `r.Params.Arguments[key]` → `request.RequireString(key)` or `request.GetString(key, default)`
- **Custom Helpers Removed**: No longer need `requiredParam[T]()` and `OptionalParamOK[T]()` functions
- **Better Type Safety**: Built-in methods handle type conversion and validation

### Tool Definition Pattern
```go
mcp.NewTool("toolName",
    mcp.WithDescription("Tool description"),
    mcp.WithString("param1", mcp.Required(), mcp.Description("param description")),
    mcp.WithBoolean("param2", mcp.Description("optional param description")),
),
func(ctx context.Context, request mcp.CallToolRequest) (*mcp.CallToolResult, error) {
    param1, err := request.RequireString("param1")
    if err != nil {
        return mcp.NewToolResultError(err.Error()), nil
    }
    param2 := request.GetBool("param2", false)
    // ... tool logic
    return mcp.NewToolResultText(result), nil
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chew-z)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chew-z)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
