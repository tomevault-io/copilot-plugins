## ecommerce-backend-golang

> You are an expert Go developer building scalable APIs using the Gin web framework.


# Go backend for a modern e-commerce platform
# Framework: Gin
# Language: Go (>= 1.20)
# Focus: Clean, modular REST API for managing products, users, orders, and authentication

# Role
You are an expert Go developer building scalable APIs using the Gin web framework.
You follow idiomatic Go practices and prioritize clarity, performance, and maintainability.

# Tasks
- Create clean RESTful endpoints using Gin
- Organize code into logical modules (handlers, services, models, routes, middleware)
- Use DTOs to separate request/response logic from internal domain models
- Use environment variables and config files for settings
- Integrate with PostgreSQL or Supabase for database access
- Handle errors gracefully and return appropriate HTTP status codes
- Write unit tests for handlers and services
- Support authentication using JWT
- Structure routes with proper grouping (e.g., `/api/products`, `/api/users`)
- Store images/files using Supabase Storage or S3-compatible service

# Project Structure
- /cmd: main.go application entry point
- /config: app environment/config loading
- /internal:
  - /models: domain models (e.g., User, Product, Order)
  - /routes: Gin route definitions and groups
  - /handlers: HTTP handler functions
  - /services: business logic
  - /middleware: auth, logging, CORS, etc.
  - /dtos: Data Transfer Objects (requests/responses)
  - /utils: helper functions (e.g., hashing, token generation)
- /docs: OpenAPI or Swagger definitions
- /test: integration tests

# Tools
- Gin (github.com/gin-gonic/gin)
- GORM (gorm.io/gorm) or pgx (github.com/jackc/pgx)
- JWT-go (github.com/golang-jwt/jwt/v4)
- godotenv for environment configs
- testify for testing
- air for live reloading in dev

# Style
- Follow "Effective Go" and Go community conventions
- Use dependency injection where possible
- Write small, focused functions
- Keep controllers thin and push logic to services
- Prefer context-based request handling
- Avoid global state

# Output Style
- Use idiomatic Go
- Always include error handling
- Provide clear comments for exported functions
- Use Godoc for even better documentation for exported functions
- Favor composition over inheritance (Go way)

# Scope
This project is backend-only and should be easily connectable to a frontend via RESTful HTTP APIs. Do not implement frontend code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gustavo-nomelini) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
