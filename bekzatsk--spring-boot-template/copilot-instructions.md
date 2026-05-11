## spring-boot-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spring Boot 4.0.5 auth template in **Kotlin** (Java 24) with JWT authentication, multi-provider social login (Google, Apple, email+password, phone+SMS OTP), push notifications (Firebase), and email (SMTP/IMAP). Uses PostgreSQL 18 with Flyway migrations and UUID v7 primary keys.

## Build & Run Commands

```bash
# Start PostgreSQL (required for local dev)
docker compose up -d

# Run application (dev profile)
./mvnw spring-boot:run

# Run all tests (H2 in-memory, no Docker needed)
./mvnw test

# Run a single test class
./mvnw test -pl . -Dtest=AccountManagementIntegrationTest

# Run a single test method
./mvnw test -pl . -Dtest=AccountManagementIntegrationTest#testChangePasswordSuccess

# Compile only
./mvnw clean compile
```

Server runs on port **7070**. Swagger UI at `/swagger-ui.html`.

## Architecture

The codebase is organized into three domain modules under `src/main/kotlin/kz/innlab/template/`:

- **authentication/** - All auth flows (local, Google, Apple, phone OTP), token management (JWT + refresh token rotation), account management (password/email/phone change), verification codes
- **notification/** - Firebase Cloud Messaging push notifications and email sending/receiving (SMTP/IMAP)
- **user/** - User entity, profile endpoint, roles, providers

Each module follows the same internal structure: `controller/`, `dto/`, `model/`, `repository/`, `service/`.

Cross-cutting concerns:
- **config/** - Security config, RSA key config, CORS, provider configs (Google, Apple, Firebase, email). All config properties are under the `app.*` namespace in `application.yaml`.
- **shared/** - `BaseEntity` (UUID v7 + `Persistable` for JPA new-entity detection), `ErrorResponse` DTO

### Key Design Decisions

- **Email is the universal identity key** for account linking. One email = one user across all providers. Phone-only users have `email = ""` with a partial unique index.
- **Refresh token rotation** with reuse detection: used tokens within 10s grace window return 409; reuse after grace revokes all user tokens.
- **SMS/Email services are interfaces** with console-logging defaults. Real providers plug in via `@Bean` (the default uses `@ConditionalOnMissingBean`).
- **Virtual threads** are enabled (`spring.threads.virtual.enabled: true`).
- **Dev profile** uses in-memory RSA keys, hardcoded verification code `123456`, and console logging for SMS/email/push.
- **Kotlin compiler plugins**: `spring` (open classes), `jpa` (no-arg constructors), `all-open` for JPA entities.

## Test Setup

Tests use H2 in-memory DB with Flyway disabled and `create-drop` DDL. External dependencies are mocked with `@MockitoBean` (GoogleIdTokenVerifier, AppleJwtDecoder, SmsService, EmailService). Email integration tests use GreenMail (in-process SMTP/IMAP) and Awaitility for async assertions.

## Database Migrations

Flyway migrations in `src/main/resources/db/migration/` (V1 through V4). Dev profile has `clean-on-validation-error: true`; prod uses strict validation.

## Renaming the Project

```bash
./scripts/rename-project.sh <new_package> <new_project_name>
# e.g., ./scripts/rename-project.sh com.innlab.cakeup cakeup
```

---
> Source: [bekzatsk/spring-boot-template](https://github.com/bekzatsk/spring-boot-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
