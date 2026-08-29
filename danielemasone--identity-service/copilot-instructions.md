## identity-service

> Identity Service API is a production-style backend portfolio project for user identity management. It demonstrates a Java 21 / Spring Boot 4 service built with an API-first contract, PostgreSQL persistence, Flyway migrations, Docker Compose runtime, Testcontainers integration tests, JaCoCo reporting, and GitHub Actions deployment to GitHub Pages.

# Codex Guide

## Project Purpose

Identity Service API is a production-style backend portfolio project for user identity management. It demonstrates a Java 21 / Spring Boot 4 service built with an API-first contract, PostgreSQL persistence, Flyway migrations, Docker Compose runtime, Testcontainers integration tests, JaCoCo reporting, and GitHub Actions deployment to GitHub Pages.

## Architecture

The service follows a small layered architecture:

```text
Client/API Consumer
  -> OpenAPI contract
  -> generated Spring MVC interface
  -> controller
  -> service
  -> domain model
  -> repository
  -> PostgreSQL
```

Supporting concerns:

* Flyway owns database schema migrations in `src/main/resources/db/migration`.
* MapStruct owns API/domain mapping implementations.
* Spring MVC controllers stay thin and delegate to services.
* Business rules live in `service`.
* Persistence logic stays behind Spring Data repositories.
* RFC 7807 error translation is centralized in `GlobalExceptionHandler`.

## Package Structure

```text
src/main/java/com/dmasone/identity
  api/controller        OpenAPI-generated interface implementations
  api/mapper            MapStruct mapper definitions
  domain/model          Internal JPA domain entities and enums
  domain/repository     Spring Data persistence ports
  infrastructure        Cross-cutting configuration and exception handling
  service               Application business logic

src/main/resources
  openapi               Source OpenAPI contract
  db/migration          Flyway SQL migrations

src/test/java
  controller            Web-layer tests with MockMvc
  service               Unit tests for business behavior
  integration           Testcontainers PostgreSQL integration tests
```

## Generated Code

OpenAPI and MapStruct code is generated during Maven builds and must not be committed or edited manually.

Expected generated locations:

```text
target/generated-sources/openapi
target/generated-sources/annotations
target/openapi-docs
```

Edit the source contract or mapper interfaces instead:

* OpenAPI source: `src/main/resources/openapi/identity-api.yaml`
* MapStruct source: `src/main/java/com/dmasone/identity/api/mapper/UserMapper.java`

Maven also generates static OpenAPI HTML documentation from `identity-api.yaml` under `target/openapi-docs`; GitHub Pages publishes it at `/openapi/`. Do not manually edit generated OpenAPI documentation.

## Build And Test Commands

Use validation-oriented commands:

```bash
mvn clean compile
mvn clean verify
```

`mvn clean verify` runs unit tests, web-layer tests, Testcontainers integration tests, packaging, and JaCoCo report generation. Docker must be running for integration tests.

Avoid recommending skipped-test flows as normal validation. If a skip is needed for a narrow local experiment, explain why.

## Docker Local Runtime

Start PostgreSQL and the application:

```bash
docker compose up --build
```

Start only PostgreSQL for local Maven runs:

```bash
docker compose up -d db
mvn spring-boot:run
```

The application expects:

```text
DB_USERNAME=postgres
DB_PASSWORD=postgres
```

Swagger UI is local-only and is available when the app is running:

```text
http://localhost:8080/api/swagger-ui.html
```

## Coverage And Documentation

Generate coverage with:

```bash
mvn clean verify
```

Local reports:

```text
target/site/jacoco/index.html
target/site/jacoco/jacoco.xml
```

Generated OpenAPI classes and generated MapStruct implementations are excluded from coverage.

Generate the Maven documentation site after verification with:

```bash
mvn site
```

Documentation ownership and public paths are:

```text
README.md                         concise repository entry point
docs/index.html                  public dashboard source (/)
docs/user-guide/*.html           public User Guide source (/user-guide/)
target/openapi-docs              generated OpenAPI HTML (/openapi/)
target/site/jacoco               generated coverage (/coverage/)
target/site                      generated Maven site (/maven-site/)
```

The dashboard and User Guide are direct static HTML sources and reuse `docs/styles.css` and `docs/script.js`. Keep them responsive, keyboard accessible, and usable without JavaScript for content or navigation. Do not add a frontend framework or documentation build chain without a concrete need.

GitHub Actions publishes these surfaces with the official Pages configure/upload/deploy actions. GitHub Pages must use `Source: GitHub Actions`.

## Testing Expectations

Keep tests meaningful and behavior-focused:

* controller tests should verify HTTP status, payloads, validation errors, and service interactions
* service tests should verify business rules such as duplicate email checks, password hashing, updates, and soft deletes
* repository and integration tests should use real PostgreSQL through Testcontainers
* avoid empty context-load tests and tests without assertions

## Codex Workflow

This repository is structurally stable. Prefer small, focused changes that preserve the current architecture.

`README.md` is the concise repository entry point, the dashboard is the public landing page, and detailed setup, runtime, testing, reporting, and troubleshooting instructions live in the HTML pages under `docs/user-guide/`.

* For backend behavior changes, update or add tests and run `mvn clean verify`.
* For API behavior changes, update `src/main/resources/openapi/identity-api.yaml` first, then implementation, tests, README, user guide, and dashboard links if affected.
* For Docker changes, run `docker compose config` and, when feasible, `docker compose up --build`.
* For documentation-only changes, keep README, `docs/user-guide/`, dashboard, Maven site references, workflow paths, and this guide consistent.
* Do not add speculative abstractions, new frameworks, or machine-specific Codex configuration.
* Prefer Spring Boot managed dependency versions unless a direct override is clearly justified and verified.

## Quality Bar

Keep the project compact, production-style, and recruiter-friendly: clear API contract, thin controllers, tested business behavior, real PostgreSQL integration tests, validated Docker runtime, generated coverage, and a responsive, accessible GitHub Pages documentation portal.

## Do Not Edit Manually

Do not manually edit:

* `target/generated-sources/openapi/**`
* `target/generated-sources/annotations/**`
* generated OpenAPI models/interfaces
* generated OpenAPI HTML documentation under `target/openapi-docs/**`
* generated MapStruct implementations
* Maven output under `target/`
* assembled Pages output under `pages/`

## Do Not Commit

Keep local and generated artifacts out of Git:

* `.env`
* `target/`
* `pages/`
* `.idea/`, `.vscode/`, `.vs/`
* logs, temp files, local build artifacts
* secrets or machine-specific configuration

When changing behavior, update the OpenAPI contract, implementation, tests, README, HTML User Guide, dashboard, and this guide when they are affected.

---
> Source: [DanieleMasone/identity-service](https://github.com/DanieleMasone/identity-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
