## journey-orchestrator

> ﻿# journey-orchestrator Development Guidelines

﻿# journey-orchestrator Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-04-10

## Active Technologies
- Java 21 (from constitution) + Spring Boot 4.0.3, Spring Data MongoDB, MongoDB Java Driver (002-mongodb-persistence)
- MongoDB (replacing in-memory storage) (002-mongodb-persistence)
- Java 21 + Spring Boot 4.0.3, MongoDB, OpenAPI 3.0.3, Lombok (004-transition-history)
- MongoDB (primary persistence layer) (004-transition-history)
- Java 21 + Spring Boot 4.0.3, SLF4J, Logback, Spring AOP (005-execution-observability)
- MongoDB (existing) (005-execution-observability)
- Java 21 (LTS) + Spring Boot 4.0.3, MongoDB, Lombok, OpenAPI 3.0.3 (006-conditional-transitions)
- MongoDB for journey definitions and instances (006-conditional-transitions)
- Java 21 (LTS) + RestAssured, Testcontainers, JUnit 5, Spring Boot Test, MongoDB Testcontainers (007-e2e-journey-tests)
- MongoDB (via Testcontainers for testing) (007-e2e-journey-tests)
- Java 21 (LTS) + Spring Boot 4.0.3, MongoDB, OpenAPI 3.0.3, Lombok, Maven, RestAssured 5.4.0, Testcontainers 1.19.7, JUnit 5 (008-graph-evolution-refactor)

- Java 21 (LTS) + Spring Boot 4.0.3, Spring Web, Spring Validation, Lombok (001-error-handling)

## Project Structure

```text
backend/
frontend/
tests/
```

## Commands

# Add commands for Java 21 (LTS)

## Code Style

Java 21 (LTS): Follow standard conventions

## Recent Changes
- 008-graph-evolution-refactor: Added Java 21 (LTS) + Spring Boot 4.0.3, MongoDB, OpenAPI 3.0.3, Lombok, Maven, RestAssured 5.4.0, Testcontainers 1.19.7, JUnit 5
- 007-e2e-journey-tests: Added Java 21 (LTS) + RestAssured, Testcontainers, JUnit 5, Spring Boot Test, MongoDB Testcontainers
- 006-conditional-transitions: Added Java 21 (LTS) + Spring Boot 4.0.3, MongoDB, Lombok, OpenAPI 3.0.3


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuscaDevs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
