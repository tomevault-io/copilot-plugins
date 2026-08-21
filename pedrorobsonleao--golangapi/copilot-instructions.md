## golangapi

> This document contains foundational mandates, architecture patterns, and conventions for the **Golang API REST** project. These instructions take absolute precedence over general defaults or workflows in all future LLM agent sessions.

# Foundational Mandates & Project Guidelines (GEMINI.md)

This document contains foundational mandates, architecture patterns, and conventions for the **Golang API REST** project. These instructions take absolute precedence over general defaults or workflows in all future LLM agent sessions.

---

## 📚 Documentos Relacionados do Projeto

Acesse outros guias e documentações do projeto nos links abaixo:

- 📖 [**README.md** (Visão Geral e Arquitetura do Projeto)](README.md)
- 🚀 [**ANTIGRAVITY.md** (Guia de Reprodução Automática)](ANTIGRAVITY.md)

---

## 1. Project Overview & Technologies

This is a high-performance REST API developed in **Go (Golang)**, designed as a modern, contract-first migration from a reference Java Spring Boot API. It manages individuals (pessoas) and validates business contracts.

### Technical Stack:
- **Language**: Go (`1.26.4` or higher)
- **Web Framework**: Labstack Echo v4 (`github.com/labstack/echo/v4`)
- **API Code Generation**: `oapi-codegen` runtime (`github.com/oapi-codegen/runtime`)
- **Database Access**: Standard `database/sql` using the MySQL/MariaDB Go Driver (`_ "github.com/go-sql-driver/mysql"`)
- **Security & JWT**: JWT-Go v5 (`github.com/golang-jwt/jwt/v5`) using asymmetric **RSA key pairs (RS256)** generated dynamically in memory at startup
- **Database Engine**: MariaDB / MySQL
- **Integration Tests**: Newman (Postman API Contract Runner)

---

## 2. Directory Layout & Module Structure

The project follows a clean, single-package architecture (excluding the generated API contract) centered under the `src/` directory to maximize simplicity, ease of testing, and maintainability:

```
golangapi/
├── db/                        # Database secrets and root passwords (e.g., pwd.txt)
├── newman/                    # Postman collection scripts, environments, and HTML test reports
│   └── tests/                 # Collection JSON and local/docker environments
├── src/                       # Complete Go source codebase (package main)
│   ├── api/                   # Isolated folder containing generated files (package api)
│   │   └── api.gen.go         # Boilerplate structs and interface generated from OpenAPI
│   ├── main.go                # Application startup, DB retry loop, RSA generation, routes, and middleware
│   ├── openapi.yaml           # Embedded OpenAPI specification for Swagger UI documentation
│   ├── server.go              # Echo HTTP handlers implementation (implements api.ServerInterface)
│   ├── server_test.go         # Robust handler unit tests utilizing the MockStore
│   └── store.go               # SQL Database Access Layer (implementing Store interface)
├── .env                       # Local environment variables (ignored by git)
├── Dockerfile                 # Multi-stage lightweight Alpine build file for container execution
├── docker-compose.yml         # Local stack orchestration: MariaDB, phpMyAdmin, Go App, and Newman
├── Makefile                   # Local task orchestration (compiling, testing, and coverage)
└── README.md                  # Human-readable documentation and architectural diagram
```

---

## 3. Foundational Development Conventions

All future enhancements and bug fixes must rigorously adhere to these technical standards:

### A. Code Layout & Package Separation
- **Do NOT mix packages inside `src/`**: All custom files in `src/` (such as `main.go`, `server.go`, `store.go`, etc.) must belong to `package main`.
- **OpenAPI Code Generation**: The auto-generated boilerplate `api.gen.go` is isolated inside `src/api/` under `package api`.
- **Imports**: Custom files must import the generated package via `"github.com/pedrorobsonleao/golangapi/src/api"`.

### B. Database Access & Schema Initialization
- **Repository Pattern**: Always define access interfaces (like `Store` in `store.go`) to allow robust mock-based testing of your handlers without relying on active databases.
- **Auto-Initialization**: The application must automatically initialize its database schema on startup. Place any required `CREATE TABLE IF NOT EXISTS ...` operations inside `main.go` right after the database connection is successfully verified.
- **Robust Connection Retry**: Database containers in Docker take a few seconds to boot up. Always maintain a retry loop (e.g., 15 attempts with a 2-second sleep delay) when establishing connections via `sql.Open`.

### C. Error Handling & Layer Decoupling
- **Decouple SQL from HTTP**: Database query methods must never leak engine-specific errors (e.g., `sql.ErrNoRows`) to the HTTP controllers. Catch them at the database access layer (`store.go`) and map them to domain errors (such as `ErrNotFound`).
- **HTTP Status Mapping**: The HTTP controller layer (`server.go`) must catch domain errors and map them to exact HTTP statuses (e.g., map `ErrNotFound` to `404 Status Not Found`).

### D. Security & Encryption
- **Asymmetric JWT Verification**: Non-public routes must be protected using asymmetric RS256 JWT validation. Generate a 2048-bit RSA key pair in-memory at startup (`main.go`) to sign/verify tokens dynamically without persisting secrets.
- **JWT Middleware**: Guard routes with middleware checking for `Authorization: Bearer <token>`. Ensure you enforce the signature verification method check (`SigningMethodRSA`) in your claims parser to prevent algorithm evasion attacks.

### E. Documentation & Comments
- **Human-centric Comments**: Document all exported functions, types, and interfaces using standard GoDoc comments.
- **Explain *Why*, not *What***: Add internal comments within complex parts (e.g., the JWT parsing logic or retry connection blocks) focusing on the technical rationale and security measures rather than simply restating the instructions.

### F. Swagger UI & Contract Serving
- **Swagger UI**: Serve the standard interactive Swagger UI HTML at `/swagger-ui`.
- **OpenAPI Contract**: Serve the embedded raw OpenAPI 3.0 contract at `/swagger-ui/openapi.yaml`.
- **Middleware Exemption**: Ensure all routes matching `/swagger-ui` (or starting with it) bypass JWT authentication middleware to remain publicly accessible.

---

## 4. Key Building, Running & Testing Commands

Developers and AI agents should leverage the **`Makefile`** to interact with the project locally:

```bash
# 1. Regeneate Go code from the OpenAPI Schema
make generate

# 2. Compile the local binary file ("api-server")
make build

# 3. Run the Go application locally
make run

# 4. Run the complete unit test suite
make test

# 5. Generate and open a local HTML code coverage report
make coverage

# 6. Clean all temporary build artifacts and test report files
make clean
```

### Docker Compose Integration

To run the complete ecosystem (MariaDB database, phpMyAdmin, the Go HTTP App) and trigger the Newman integration tests of contract:

```bash
# Stop conflicting containers, build and run the services
docker compose down
docker compose up --build
```
*Wait for `api_tests` (Newman) to finish with status code `0`.*

---

## 5. Git and CI Integrity
- **Secrets protection**: Never track or commit `.env` or files inside `db/`.
- **Relatório ignora**: Local `.html` files (like `coverage.html` or files inside `newman/tests/newman/`) are ignored via `.gitignore` to prevent repository pollution.
- **No force staging**: Always add specific changed files individually (e.g., `git add src/server.go`) rather than running `git add .`.

---
> Source: [pedrorobsonleao/golangapi](https://github.com/pedrorobsonleao/golangapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
