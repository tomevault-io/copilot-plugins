## api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the BAB-INSA API project - a REST API for a foosball (baby foot) association built with Go using the Gin framework and JWT authentication. The project uses Go 1.22.2 and follows standard Go project structure for web API development.

## Commands

- **Run the application**: `go run main.go`
- **Build the application**: `go build`
- **Format code**: `go fmt ./...`
- **Run tests**: `go test ./...`
- **Install dependencies**: `go mod tidy`

### Code Quality Tools

- **Lint code**: `golangci-lint run`
- **Static analysis**: `staticcheck ./...`
- **Security scan**: `gosec ./...`
- **Vet code**: `go vet ./...`
- **Format with simplifications**: `gofmt -s -w .`

**Install quality tools:**
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install honnef.co/go/tools/cmd/staticcheck@latest
go install github.com/securego/gosec/v2/cmd/gosec@latest
```

## Architecture

This is the BAB-INSA foosball association API with:
- `main.go`: Entry point for the Gin web server
- `go.mod`: Go module definition with Go 1.22.2 requirement
- Authentication system for managing association members
- Database integration for tracking players, games, and tournaments

**Key Dependencies:**
- `github.com/gin-gonic/gin`: Web framework for building REST APIs
- `github.com/golang-jwt/jwt/v5`: JWT implementation for authentication
- Database packages for PostgreSQL with GORM
- Additional middleware for foosball-specific features

**Project Structure:**
- `/packages/auth`: Authentication module for association members
- `/handlers`: HTTP request handlers for foosball features
- `/middleware`: JWT auth, CORS, roles middleware
- `/models`: Data structures for users, games, tournaments
- `/migrations`: Database migrations system
- `/config`: Database and application configuration
- `/docs`: Swagger API documentation

The project follows standard Go web API conventions with JWT-based authentication tailored for a foosball association management system.

---
> Source: [BAB-INSA/api](https://github.com/BAB-INSA/api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
