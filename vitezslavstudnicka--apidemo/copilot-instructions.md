## apidemo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot Kotlin application that uses:
- Kotlin 2.2.21
- Spring Boot 4.0.2
- PostgreSQL database with Flyway migration support
- Spring Data JPA for database access
- Spring Security for security
- Testcontainers for integration testing
- Docker Compose for local development

## Key Dependencies

- Spring Boot Starter Web MVC
- Spring Boot Starter Data JPA
- Spring Boot Starter Security
- Spring Boot Starter Actuator
- Spring Boot Starter Flyway
- PostgreSQL JDBC Driver
- Testcontainers for PostgreSQL testing

## Development Commands

### Build the project
```bash
./gradlew build
```

### Run the application
```bash
./gradlew bootRun
```

### Run tests
```bash
./gradlew test
```

### Run a single test class
```bash
./gradlew test --tests "ApiDemoApplicationTests"
```

### Run a single test method
```bash
./gradlew test --tests "ApiDemoApplicationTests.contextLoads"
```

### Clean and rebuild
```bash
./gradlew clean build
```

## Docker Setup

The project includes a `compose.yaml` file that defines a PostgreSQL service:
```yaml
services:
  postgres:
    image: 'postgres:latest'
    environment:
      - 'POSTGRES_DB=mydatabase'
      - 'POSTGRES_PASSWORD=secret'
      - 'POSTGRES_USER=myuser'
    ports:
      - '5432'
```

## Testing

Tests use Testcontainers to spin up PostgreSQL containers for integration testing. The test configuration is defined in `TestcontainersConfiguration.kt`.

## Database Migration

Flyway is configured for database migrations. Migration scripts should be placed in `src/main/resources/db/migration`.

## Architecture

The application follows standard Spring Boot conventions:
- Main application class: `ApiDemoApplication.kt`
- Test configuration with Testcontainers: `TestcontainersConfiguration.kt`
- Test classes in `src/test/kotlin/org/example/apidemo/`
- Main source code in `src/main/kotlin/org/example/apidemo/`
- Resources in `src/main/resources/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VitezslavStudnicka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
