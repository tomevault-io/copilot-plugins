## transportation-project

> This project is a comprehensive **Transportation Management System** built with Go. It is designed to manage the logistics and operations of a transportation business, covering a wide range of functionalities from trip management to financial tracking.

# Project Context: Transportation Management System

#Listen without my permission do not write a single line of code, if i told work read only mode just give me plan and works as a read only mode, then if i told you to write code then only you can write code.

## Overview
This project is a comprehensive **Transportation Management System** built with Go. It is designed to manage the logistics and operations of a transportation business, covering a wide range of functionalities from trip management to financial tracking.

### Key Modules
-   **Trips**: Core functionality for managing transportation trips, including "Outside Trips" and "Own Vehicle Trips".
-   **Fleet Management**: Management of **Vehicles**, **Drivers**, and **Helpers**.
-   **Clients**: Handling **Customers** and **Dealers**.
-   **Financials**: **Billing**, **Route Pricing** (calculating rates based on dealer/destination), **Daily Side Cash**, and **Purchases**.
-   **Authentication**: JWT-based authentication system with provisions for Google OAuth.
-   **Media**: Image uploads using Cloudinary.

## Tech Stack
-   **Language**: Go (Golang) v1.24+
-   **Web Framework**: [Gin](https://github.com/gin-gonic/gin)
-   **Database**: PostgreSQL
-   **ORM**: [GORM](https://gorm.io/)
-   **Configuration**: `godotenv` for environment variables.
-   **Other Libraries**: `jwt-go` for auth, `cloudinary-go` for media.

## Architecture
The project follows a standard modular architecture within the `internal/` directory. Each domain (e.g., `customer`, `trip`, `auth`) is self-contained with the following structure:
-   **`handler.go`**: HTTP request handlers (Controllers).
-   **`service.go`**: Business logic layer.
-   **`repo.go`**: Database access layer (Repository pattern using GORM).
-   **`model.go`**: Database entities/structs.
-   **`dto.go`**: Data Transfer Objects for API requests/responses.
-   **`route.go`**: API route definitions.

## Configuration
The application is configured using environment variables.
1.  Copy `.env.example` to `.env`:
    ```bash
    cp .env.example .env
    ```
2.  Configure the following variables (and others as needed):
    -   `DATABASE_URL`: Connection string for PostgreSQL (e.g., `postgres://user:pass@localhost:5432/transportation?sslmode=disable`).
    -   `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`: For OAuth (if enabled).
    -   Cloudinary credentials (likely required if using image uploads).

## Building and Running

### Prerequisites
-   Go 1.24 or higher
-   PostgreSQL running locally or accessible via network.

### Commands
-   **Install Dependencies**:
    ```bash
    go mod tidy
    ```
-   **Run the Application**:
    ```bash
    go run main.go
    ```
    The server typically starts on port **8080**.

-   **Build Binary**:
    ```bash
    go build -o transportation_app main.go
    ```

### Database Migrations
Migration files are located in the `migrations/` directory in standard SQL format (`.up.sql`, `.down.sql`).
-   These are likely intended to be run with a tool like [golang-migrate](https://github.com/golang-migrate/migrate).
-   **Example command** (assuming `migrate` is installed):
    ```bash
    migrate -path migrations -database "$DATABASE_URL" up
    ```

## Development Conventions
-   **Dependency Injection**: Dependencies (repos, services) are injected manually in `main.go`.
-   **Routing**: Routes are defined in each module's `SetupRoutes` function and aggregated in `main.go`.
-   **Error Handling**: specific error handling conventions are generally found in the `service` layer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mehedi19087) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
