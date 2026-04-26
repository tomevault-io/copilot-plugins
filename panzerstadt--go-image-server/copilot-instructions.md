## go-image-server

> - Build: `go build -o go-image-server *.go`

# Go Image Server - Development Guide

## Build & Run Commands
- Build: `go build -o go-image-server *.go`
- Run: `./go-image-server` or `./start.sh` (runs as background service)
- Stop service: `./stop.sh`
- View logs: `./logs.sh` (uses tmux if available)

## Code Style Guidelines
- Format: `gofmt -w -s *.go` (simplify code when possible)
- Imports: Standard library first, then third-party packages with blank line separator
- Naming: Use snake_case for variables/functions, camelCase for struct fields
- Error handling: Check errors immediately, use idiomatic `if err != nil` pattern
- Types: Define structs near their primary usage
- Comments: Add comments for non-obvious functions and complex code sections
- Goroutines: Always use proper synchronization (WaitGroups, channels)
- Imports order: standard library first, then external packages
- Error handling: Return errors rather than logging when possible

## Project Structure
- Place new HTTP handlers in appropriate semantic files
- Add monitoring code in uptime.go
- Put utility functions in appropriate domain files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/panzerstadt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
