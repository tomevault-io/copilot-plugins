## pokerhans

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Build & Run Commands
- Build: `go build -o app ./cmd/pokerhans`
- Run server: `go run ./cmd/pokerhans`
- Lint: `golangci-lint run`
- Test: `go test ./...`
- Single test: `go test ./path/to/package -run TestName`

## Code Style Guidelines
- **Formatting**: Use `gofmt` for all Go code
- **Imports**: Group standard library imports first, then third-party, then internal
- **Error Handling**: Always check and handle errors; use descriptive error messages
- **Types**: Use strong typing; avoid interface{} when possible
- **Naming**: 
  - Use camelCase for variables, PascalCase for exported functions/types
  - Use short but descriptive names
- **Database**: Always use parameterized queries to prevent SQL injection
- **Frontend**: Follow Tailwind CSS conventions for all styling
- **Documentation**: Document all exported functions and types with comments

---
> Source: [klausbreyer/pokerhans](https://github.com/klausbreyer/pokerhans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
