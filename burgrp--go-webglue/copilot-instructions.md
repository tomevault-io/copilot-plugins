## go-webglue

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

go-webglue is a Go web framework that provides seamless integration between Go backend APIs and JavaScript frontend code. It automatically bridges Go struct methods to JavaScript API calls, handles Server-Sent Events (SSE) for real-time updates, and serves static resources with automatic minification and import map generation.

## Architecture

### Core Components

**Module System** (mux_handler.go:8-48)
- Modules are the primary building blocks, each containing:
  - `Name`: Module identifier used in URLs and JS API
  - `Resources`: Embedded filesystem (`embed.FS`) containing client-side files
  - `Events`: Array of SSE events that can be emitted from Go
  - `Api`: Go struct with exported methods exposed as HTTP endpoints

**Handler Chain** (mux_handler.go:20-48)
- `NewHandler()` creates an `http.ServeMux` with three mounted handlers:
  - `/` → `StaticHandler`: Serves HTML, CSS, JS with minification
  - `/api/` → `ApiHandler`: Routes API calls to Go methods
  - `/events` → SSE handler: Streams server events to clients

### Request Flow

**API Calls** (api_handler.go:53-208)
1. Client calls `api.moduleName.functionName(...params)`
2. Request hits `/api/{moduleName}/{functionName}`
3. ApiHandler uses reflection to:
   - Find the matching Go method (capitalizes first letter)
   - Inject typed parameters from context (e.g., `context.Context`, `*http.Request`)
   - Unmarshal remaining JSON parameters from request body
   - Call the Go method with all parameters
4. Return values are marshaled as `{"result": value}` or `{"error": "message"}`

**CallChecker Interface** (api_handler.go:36-38)
- Optional interface for API structs to implement authentication/authorization
- `CheckCall(request *http.Request, functionName string) ([]any, error)`
- Returns additional typed parameters injected into the function call
- Example in test/main.go:30-34 shows extracting User-Agent header

**Event System** (event_handler.go)
- Events created with `webglue.NewEvent(name)`
- Call `event.Emit(params...)` from Go to broadcast to all connected clients
- Client receives events via SSE and triggers jQuery events like `webglue-{module}-{event}`
- Special "webglue.tick" event fires every second on all DOM elements

### Static Resource Handling

**Development Mode** (static_handler.go:101-128)
- Set environment variable `{MODULENAME}_DEV=/path/to/source` (uppercase module name)
- Files served directly from filesystem without minification
- Supports hot-reload workflow

**Production Mode** (static_handler.go:130-145)
- All resources from `embed.FS` are minified and cached in memory
- Generates import map for ES modules from embedded JS files
- Injects stylesheet links and import map into `{WEBGLUE}` placeholder in HTML

**Import Map Generation** (static_handler.go:170-180)
- Scans all embedded `.js` files
- Creates import map entries mapping `"filename"` → `"./filename.js"`
- Allows imports like `import { api } from "webglue"` instead of relative paths

### Client-Side Architecture

**webglue.js** (pkg/client/webglue.js)
- Discovers available API methods via `api/webglue/discover` endpoint
- Dynamically creates `api.{module}.{function}()` methods
- Handles SPA routing with history API
- Loads page modules as `{pagename}.page.js`
- Provides tag factory functions (`DIV()`, `BUTTON()`, etc.) for jQuery-based DOM building

**Page Structure** (test/client/home.page.js)
- Export default object with:
  - `title`: Page title string
  - `render(container, page, params)`: Returns array of jQuery elements
  - `error(e)`: Optional custom error handler
  - `check(url, params)`: Optional redirect logic

**API Header Support**
- Store headers in localStorage/sessionStorage with prefix `webglue.headers.`
- Automatically sent with all API requests (webglue.js:197-209)
- Use for authentication tokens, etc.

## Development Commands

### Build and Run
```bash
# Run the test application
cd test
go run main.go

# Build the library
cd pkg
go build

# Run test app with hot reload
WEBGLUE_DEV=/path/to/pkg/client TEST_DEV=/path/to/test/client go run test/main.go
```

### Testing
No test files currently exist in the repository.

## Key Implementation Patterns

### Creating a Module
```go
//go:embed client/*
var clientResources embed.FS

type MyApi struct {
    // API state fields
}

func (api *MyApi) SomeMethod(ctx context.Context, param1 string) (string, error) {
    // Method implementation
    return "result", nil
}

options := webglue.Options{
    Modules: []*webglue.Module{{
        Name:      "mymodule",
        Resources: &clientResources,
        Events:    []*webglue.Event{myEvent},
        Api:       &MyApi{},
    }},
}
```

### Emitting Events
```go
event := webglue.NewEvent("update")
// Later...
event.Emit(data1, data2) // Variadic parameters
```

### Using Custom HTML Template
Replace `{WEBGLUE}` placeholder with custom layout:
```go
options := webglue.Options{
    IndexHtml: customTemplate, // Must contain {WEBGLUE} placeholder
    Modules:   modules,
}
```

## Dependencies

- `github.com/json-iterator/go`: Fast JSON marshaling (used instead of stdlib)
- `github.com/r3labs/sse/v2`: Server-Sent Events implementation
- `github.com/tdewolff/minify/v2`: Minification for HTML/CSS/JS/JSON/XML/SVG
- jQuery: Bundled in pkg/client/jquery.js, used for DOM manipulation

## Notes

- API method names are automatically converted: Go `MyMethod` becomes JS `myMethod`
- The framework always includes a "webglue" core module for discovery
- Static handler serves index.html for any non-existent path (SPA fallback)
- Error returns from Go methods automatically become `{"error": "..."}` responses
- Multiple return values become arrays in JS; single values are unwrapped

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/burgrp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/burgrp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
