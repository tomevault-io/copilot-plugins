## wsshell

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

wsShell is a lightweight desktop SSH client built with Wails v2 (Go backend + React frontend). It provides SSH terminal, SFTP file management, and VNC remote desktop functionality for managing multiple servers.

## Build and Development Commands

### Development Mode
```bash
# Run in development mode with hot reload
wails dev
```

### Build
```bash
# Build production binary
wails build
```

### Frontend Only
```bash
cd frontend
npm install          # Install dependencies
npm run dev          # Start Vite dev server
npm run build        # Build for production
npm run lint         # Run ESLint
```

### Testing
```bash
# Run Go tests
go test ./...

# Run specific package tests
go test ./internal/ssh
go test ./internal/sftp
go test ./internal/config
```

## Architecture

### High-Level Structure

This is a Wails v2 application using a **Go backend** with **embedded React frontend**:

```
┌─────────────────────────────────────────────────────┐
│  Desktop Application (Wails Runtime)                │
├──────────────────┬──────────────────────────────────┤
│   Go Backend     │   React Frontend (embedded)     │
│                  │                                  │
│  - SSH Service   │  - Terminal (xterm.js)          │
│  - SFTP Manager  │  - File Manager                 │
│  - Config Manager│  - VNC Viewer (noVNC)           │
│  - SQLite Store  │  - Server List                  │
│                  │                                  │
└──────────────────┴──────────────────────────────────┘
         │                        │
         └────────┬───────────────┘
                  │
    Wails Bindings (Request-Response)
    Wails Events (Streaming)
```

### Backend Modules (`internal/`)

- **`internal/ssh/`** - SSH connection and session management
  - Maintains concurrent SSH sessions with stdin/stdout streaming
  - Supports password and private key authentication
  - Uses PTY for interactive shell sessions
  - Emits events: `ssh:{sessionId}:stdout`, `ssh:{sessionId}:stderr`

- **`internal/sftp/`** - SFTP file transfer operations
  - Reuses SSH connections for SFTP subsystem
  - File upload/download with progress streaming
  - Local and remote file system operations
  - Emits events: `sftp:upload:progress`, `sftp:download:progress`

- **`internal/config/`** - Server configuration management
  - CRUD operations for server configurations
  - Encrypts credentials before storage (AES-256-GCM)
  - Server grouping and favorites
  - SQLite persistence via `internal/store/`

- **`internal/crypto/`** - Encryption utilities
  - AES-256-GCM encryption for passwords/keys
  - Master key derivation from OS keychain (Windows DPAPI/macOS Keychain)

- **`internal/store/`** - Database layer
  - SQLite with migrations
  - Stores encrypted server configurations

### Frontend Structure (`frontend/src/`)

- **`components/`** - React components
- **`stores/`** - Zustand state management
- **`types/`** - TypeScript type definitions
- **`assets/`** - Static assets and app icons

### Communication Patterns

1. **Wails Bindings** (Request-Response)
   - Go methods exported to frontend via `Bind[]` in `main.go`
   - Example: `app.sshService.Connect()`, `app.configManager.GetServers()`
   - Frontend calls these methods directly and receives typed responses

2. **Wails Events** (Server-to-Client Streaming)
   - Go emits events via `runtime.EventsEmit(ctx, eventName, data)`
   - Frontend subscribes: `EventsOn(eventName, callback)`
   - Used for: terminal output, file transfer progress, VNC frames

## Key Design Decisions

- **Session Management**: SSH sessions use goroutines with channels for stdin/stdout streaming to avoid blocking
- **Concurrency**: All service managers use `sync.RWMutex` for thread-safe access to session/client maps
- **Security**: Credentials are encrypted at rest using AES-256-GCM with OS-derived master keys
- **Session IDs**: Format `{username}@{host}:{port}` for easy identification
- **File Transfer**: 32KB buffer size for streaming uploads/downloads with progress events

## Common Patterns

### Adding a New Backend Service

1. Create service in `internal/{service}/`
2. Implement `SetContext(ctx context.Context)` for Wails events
3. Export in `main.go` `Bind[]` array
4. Use response structs: `{Success bool, Error string, ...}`

### Adding Event Streaming

```go
// In Go service method
if m.ctx != nil {
    runtime.EventsEmit(m.ctx, "event:name", data)
}

// In React component
useEffect(() => {
    const unlisten = EventsOn("event:name", (data) => {
        // handle event
    });
    return () => { unlisten(); };
}, []);
```

### Error Handling

- Return error tuples: `(Response, error)` for Wails-bound methods
- Use `Success bool` and `Error string` fields in response structs
- Always close client connections on errors to avoid resource leaks

## Dependencies

- **Go**: `wails v2.12.0`, `golang.org/x/crypto/ssh`, `pkg/sftp`, `mattn/go-sqlite3`
- **Frontend**: `React 18`, `TypeScript`, `TailwindCSS`, `Zustand`, `xterm.js`, `noVNC`

## Project Status

This project is in **Phase 1 (Core)** - SSH terminal, SFTP file management, and server configuration are implemented. VNC functionality is planned for Phase 2.

---
> Source: [Julyos-rgb/wsShell](https://github.com/Julyos-rgb/wsShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
