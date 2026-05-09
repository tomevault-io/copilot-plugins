## claude-serve

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Go HTTP server wrapping Claude Code CLI as Ollama and OpenAI compatible API.

## Structure

```
main.go        # Entry, signal handling, jwt subcommand
server.go      # HTTP handlers, Ollama + OpenAI API routes
websocket.go   # WebSocket handlers: claude-stream, shell, sessions
claude.go      # CLI wrapper, cmdReader for streaming
config.go      # Env loading via dotenv
auth.go        # JWT authentication middleware
```

## Development

```sh
make build     # Debug binary → ./tmp/main
make release   # Stripped binary → ./main
make install   # Install as /usr/local/bin/claude-serve
make test      # Fast unit tests (-short)
make smoke     # All tests including e2e
make lint      # go vet + staticcheck
make run       # Build and run server
```

Run single test: `go test -v -run TestName ./...`

## Naming

Use descriptive names: `server`, `reader`, `session`.
Exception: HTTP handlers use `w`, `r` (idiomatic Go), loop indices (`i`, `x`).

## Key Patterns

**Streaming**: `exec.Cmd.StdoutPipe()` with `scanChunks()` splitter.
Splits on `.` `!` `?` `\n` for natural chunk boundaries.

**Context**: All handlers pass request context to ClaudeRunner for cancellation.

**Temp dirs**: Each request creates temp dir in CLAUDE_WORKING_DIR, cleaned via
`cmdReader.Close()`. WebSocket sessions use `claude-ws-*` prefix, cleaned on
`ClaudeSession.Close()`.

**Error handling**: `ClaudeError` wraps `exec.ExitError` with stderr message.

**Auth**: `AuthMiddleware` validates JWT (HS256) when `JWT_SECRET` is set.
Empty secret disables auth entirely.

**CORS**: `CORSMiddleware` wraps auth. Dev mode allows all origins,
production restricts to same-host. WebSocket uses same origin check.

## API Routes

**Ollama** (`/api/*`): chat, generate, tags, version
**OpenAI** (`/v1/*`): chat/completions (SSE streaming), models
**Claude Code**: sessions, active, claude-stream (WS ?resume=ID), shell (WS)
**Docs**: `/docs/` (Swagger UI), `/openapi.json`

Not implemented: model management, embeddings, keep-alive.

## Testing

E2E tests in `tests/e2e_test.go`. Mark slow tests:
```go
if testing.Short() { t.Skip("e2e") }
```

Unit tests are `*_test.go` next to source files.

---
> Source: [kronael/claude-serve](https://github.com/kronael/claude-serve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
