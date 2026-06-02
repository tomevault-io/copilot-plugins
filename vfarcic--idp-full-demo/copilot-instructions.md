## idp-full-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- Run locally: `go run .`
- Build: `go build -o ./tmp/main .`
- Hot reload: `air`
- Docker image: `./platform build image [TAG] --registry ghcr.io/vfarcic --image idp-full-app`

## Test Commands
- Run unit tests: `go test -tags=unit ./...`
- Run integration tests: `go test -tags=integration ./...`
- Run single unit test: `go test -tags=unit -run "TestRootHandler" ./...`
- Run single integration test: `go test -tags=integration -run "TestVideoPut" ./internal/handlers/...`

## Code Style Guidelines
- **Imports**: Standard library first, then third-party packages, then local packages
- **Error Handling**: Return errors early, use appropriate HTTP status codes, structured logging with slog
- **Naming**: camelCase for variables, PascalCase for exported functions/types
- **Types**: Use structs with JSON tags for data models
- **Testing**: Unit tests with `//go:build unit` tag, integration tests with `//go:build integration` tag
- **Database**: Use pgx for PostgreSQL, with connection pooling and proper cleanup

---
> Source: [vfarcic/idp-full-demo](https://github.com/vfarcic/idp-full-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
