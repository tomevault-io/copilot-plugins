## 20251115-sample-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-module Maven project demonstrating different Spring Boot application implementations with performance testing capabilities:
- `sample-web`: Traditional Spring MVC with JPA and blocking I/O
- `sample-webflux`: Reactive implementation using WebFlux and R2DBC
- `performance/`: K6-based performance testing infrastructure

Both main modules share the same API design and database schema but use different technology stacks.

## Development Commands

### Build and Package
```bash
# Build all modules
mvn clean package

# Build specific module
mvn clean package -pl sample-web
mvn clean package -pl sample-webflux
```

### Running Applications
```bash
# Start PostgreSQL database and WireMock (for EC feature testing)
docker-compose up -d

# Run sample-web (blocking)
mvn spring-boot:run -pl sample-web

# Run sample-webflux (reactive)
mvn spring-boot:run -pl sample-webflux
```

### Testing
```bash
# Run all tests
mvn test

# Run tests for specific module
mvn test -pl sample-web
mvn test -pl sample-webflux
```

### Performance Testing

The `performance/` directory contains K6 load test scripts and multiple Docker Compose configurations for different scenarios:

- `performance/backend/` - Backend services (PostgreSQL + WireMock)
- `performance/web-platform-thread/` - Platform thread configuration
- `performance/web-virtual-thread/` - Virtual thread configuration (Java 21+)
- `performance/webflux/` - WebFlux reactive configuration

Each subdirectory has its own `docker-compose.yml` for isolated testing.

### Database

- PostgreSQL database runs on port 5432
- Default credentials: postgres/secret
- Schema is auto-initialized via schema.sql

### WireMock

- WireMock server runs on port 8081
- Mock definitions are in `wiremock/mappings/`
- Response files are in `wiremock/__files/`

## Architecture

### Project Structure

- Root POM acts as parent for multi-module setup
- Each module is independently deployable Spring Boot application
- Shared package structure: `com.example.sample.{web|webflux}.{feature}`

### Module Differences

- **sample-web**: Uses Spring Web + JPA + PostgreSQL driver + RestClient
- **sample-webflux**: Uses Spring WebFlux + R2DBC + R2DBC PostgreSQL + WebClient

### Common Layer Structure

Both modules follow the same layered architecture:

- `controller/`: REST controllers exposing API endpoints
- `service/`: Business logic layer
- `repository/`: Data access layer (JPA vs R2DBC)
- `entity/`: Database entities
- `dto/`: Data transfer objects for API contracts
- `config/`: Configuration classes

### Features

#### TODO Feature (`todo/` package)

Complete CRUD API for TODO items:

- `POST /api/todos` - Create todo
- `GET /api/todos/{id}` - Get single todo
- `GET /api/todos` - Get paginated todos (max 100 per page)
- `PUT /api/todos/{id}` - Update todo
- `PATCH /api/todos/{id}/complete` - Mark todo as complete
- `DELETE /api/todos/{id}` - Delete todo

#### EC Feature (`ec/` package - sample-web only)

Demonstrates external API integration using WireMock:

- `POST /api/ec/buy` - Purchase item (calls 3 backend APIs: inventory, payment, shipping)
- Uses `RestClient` with custom configuration via `@EcBackendClient` qualifier
- Backend API URL configured via `sample.ec.backend-base-url` property

## Java Version
Both modules use Java 24 as specified in their POM files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmiya-tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
