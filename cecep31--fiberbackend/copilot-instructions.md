## fiberbackend

> This document provides a comprehensive guide for developers working on the Go Fiber Backend project. It outlines the project's architecture, development practices, and operational procedures.

# GEMINI Code Companion: Go Fiber Backend

This document provides a comprehensive guide for developers working on the Go Fiber Backend project. It outlines the project's architecture, development practices, and operational procedures.

## Project Overview

This project is a modern REST API for a blog/content management system built with the Go programming language and the Fiber web framework (v3). It leverages a PostgreSQL database with GORM for data storage and follows a clean architecture pattern, separating concerns into distinct layers: handlers, services, and repositories.

### Key Technologies

*   **Backend:** Go (version 1.25.0)
*   **Web Framework:** Fiber v3 (Note: utilizes v3 specific patterns like `c.Bind().Body(&req)`)
*   **Database:** PostgreSQL with GORM
*   **Authentication:** JWT (JSON Web Tokens)
*   **Validation:** go-playground/validator
*   **Containerization:** Docker
*   **Hot Reloading:** Air

### Architecture

The project adheres to a clean architecture (Handler → Service → Repository → Model), promoting a separation of concerns and maintainability.

*   **`cmd/main.go`**: The application's entry point. Initializes config, DI container, Fiber app, middleware, and routes. Implements graceful shutdown.
*   **`internal/`**: Core application logic.
    *   **`di/`**: Manual dependency injection container (`container.go`). *Note: Does not use reflection-based DI libraries like dig.*
    *   **`handler/`**: HTTP handlers processing requests and formulating responses using `pkg/response` helpers.
    *   **`service/`**: Business logic and use cases. Orchestrates repositories and handles transactions.
    *   **`repository/`**: Data access layer using GORM for PostgreSQL.
    *   **`model/`**: GORM models with JSON and DB tags.
    *   **`middleware/`**: Global and route-specific middleware (Auth, Rate Limiting, Logging, Security headers).
    *   **`routes/`**: API route definitions, organized by domain.
*   **`pkg/`**: Shared utility packages.
    *   **`response/`**: Standardized API response helpers (e.g., `Success`, `Created`, `BadRequest`, `HandleBindError`).
    *   **`validator/`**: Custom struct validator integration for Fiber.
*   **`config/`**: Configuration management using environment variables and `.env` files.

## Building and Running

### Prerequisites

*   Go 1.25.0+
*   PostgreSQL 14+
*   Air (for live-reloading)

### Local Development

1.  **Environment Setup:**
    ```bash
    cp .env.example .env
    # Edit .env with your local credentials. JWT_SECRET must be at least 32 characters.
    ```

2.  **Install Dependencies:**
    ```bash
    go mod download
    ```

3.  **Run the Application:**
    *   **Directly:** `go run cmd/main.go`
    *   **With Hot Reloading:** `air` (Uses `.air.toml` configuration)

4.  **Testing:**
    ```bash
    go test ./...                   # Run all tests
    go test -v ./...                # Verbose output
    go test -cover ./...            # With coverage
    ```

5.  **Linting & Quality:**
    ```bash
    golangci-lint run               # Run all linters (see .golangci.yml)
    go vet ./...                    # Vet all packages
    gofmt -l .                      # Check formatting
    ```

### TODO: Makefile
The `README.md` references a `Makefile` with commands like `make build`, `make dev`, `make test`, `make fmt`, and `make lint`, but no `Makefile` was found in the repository root. Developers should use the `go` commands listed above until a `Makefile` is implemented.

## Development Conventions

### Code Style
*   **Imports:** Group imports: 1) Standard library, 2) Local (`fiberbackend/...`), 3) Third-party. Separate with blank lines.
*   **Naming:** 
    *   Packages: lowercase, single word.
    *   Exported identifiers: `PascalCase`.
    *   Unexported identifiers: `camelCase`.
    *   Interfaces: Noun ending in "-er" (e.g., `UserService`).
*   **Formatting:** Use `gofmt`. Max line length is 140 characters (per `.golangci.yml`).

### API Practices
*   **Versioning:** All API endpoints are prefixed with `/v1` (defined in `internal/routes/routes.go`).
*   **JSON Responses:** Use `pkg/response` helpers for consistent API output.
*   **Validation:** Use `go-playground/validator` tags in request structs. Handle binding errors with `response.HandleBindError(c, err)`.
*   **Error Handling:** Return early on errors. Wrap errors with context: `fmt.Errorf("context: %w", err)`.

### Testing
*   Table-driven tests are preferred.
*   Use `github.com/stretchr/testify/assert` for assertions.
*   Tests are located in the `test/` directory and alongside source files.

## API Structure

*   **Auth:** `/v1/auth/*` (Register, Login, Refresh, Password management)
*   **Posts:** `/v1/posts/*` (CRUD, Comments, Likes, Views, Tag filtering)
*   **Users:** `/v1/users/*` (Profile, Following)
*   **Chat:** `/v1/chat/conversations/*` (Conversational AI management)
*   **Holdings:** `/v1/holdings/*` (Asset tracking)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cecep31)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cecep31)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
