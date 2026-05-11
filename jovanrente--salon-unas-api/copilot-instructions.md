## salon-unas-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
./gradlew build

# Run
./gradlew bootRun

# Tests
./gradlew test

# Single test class
./gradlew test --tests "com.salonunas.salonunasapi.ClassName"
```

## Architecture

Spring Boot + Kotlin REST API for a nail salon. Uses MySQL (`salon_unas` database, configured in `application.properties`).

**Request flow:** `Controller → Service → Repository`

Each domain entity follows this structure:
- `model/` — JPA entity (`@Entity`)
- `repository/` — Spring Data JPA interface extending `JpaRepository`
- `service/` — business logic; maps entities to/from DTOs via private extension functions
- `controller/` — REST endpoints under `/api/{resource}`
- `dto/` — `*Request` (input) and `*Response` (output) data classes; entities are never exposed directly

**DTO mapping** is done inside the Service layer using private Kotlin extension functions on the entity (e.g., `Cliente.toResponse()`). Keep this pattern when adding new entities.

**Validation** errors are caught in the controller and returned as `mapOf("error" to message)` with a 400 status. Business rule violations throw `IllegalArgumentException` from the Service.

---
> Source: [jovanrente/salon-unas-api](https://github.com/jovanrente/salon-unas-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
