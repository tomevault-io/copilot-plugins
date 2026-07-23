## racore-project

> Racore Browser project context -- Go daemon + React frontend + Tauri v2 desktop


# Racore Browser Project

## Overview

Three-tier desktop app:
- **Frontend**: `app/` -- Next.js 16 / Vinext / React / TypeScript / Tailwind
- **Backend**: `god/` -- Go daemon (`racored`), REST API on `127.0.0.1:47831`
- **Desktop**: `src-tauri/` -- Tauri v2 Rust shell

API client: `app/lib/racore-client.ts` `daemonRequest()` (single entry point for all frontend-to-daemon calls)

## Documentation

Read these files for context before making changes:

| Topic | File |
|-------|------|
| Architecture | `.llms/01-architecture.md` |
| Frontend | `.llms/02-frontend.md` |
| Go backend | `.llms/03-backend.md` |
| Tauri desktop | `.llms/04-desktop.md` |
| API reference | `.llms/05-api-reference.md` |
| Data flow | `.llms/06-data-flow.md` |
| Types | `.llms/07-types.md` |
| Build & test | `.llms/08-build-and-test.md` |

## Commands

```bash
# Build Go daemon
cd god && go build -o build/racored ./cmd/racored/

# Start daemon
RACORE_DATA_DIR=/tmp/racored-dev ./god/build/racored

# Start frontend
npm run dev

# Go tests
cd god && go test -race -count=1 ./internal/...

# All tests
npm test
```

## Conventions

- Error responses: `{"detail": "message"}` (not `{"error": "..."}`)
- JSON fields: camelCase in both Go and TypeScript
- Config: `RACORE_*` environment variables
- Transport `Close()` must `syscall.Shutdown(SHUT_RD)` before closing

---
> Source: [racore-xyz/racore-browser](https://github.com/racore-xyz/racore-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
