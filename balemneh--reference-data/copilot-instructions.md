## reference-data

> This is a full-stack application that provides a centralized service for managing reference data. It consists of a Java/Spring Boot backend, an Angular frontend, and several backing services.

# Enterprise Reference Data Service

This is a full-stack application that provides a centralized service for managing reference data. It consists of a Java/Spring Boot backend, an Angular frontend, and several backing services.

## Project Overview

The project is a multi-module Maven project. The backend is built with Java 21, Spring Boot, and Spring Cloud. It uses PostgreSQL as its database, with Liquibase for database migrations. It also integrates with Kafka for event-driven communication and Redis for caching. Keycloak is used for authentication and authorization.

The frontend is an Angular application that uses the US Web Design System (USWDS) for its UI components.

The entire application is containerized using Docker and can be orchestrated with Docker Compose.

## Building and Running

The project uses Maven for building and managing dependencies. The `Makefile` provides several useful commands for building, testing, and running the project.

### Prerequisites

*   Java 21
*   Maven
*   Docker
*   Docker Compose

### Key Commands

*   `make build`: Build all modules.
*   `make test`: Run unit tests.
*   `make test-integration`: Run integration tests.
*   `make run`: Build and run the Spring Boot applications.
*   `make up`: Start the Docker Compose infrastructure.
*   `make down`: Stop the Docker Compose infrastructure.
*   `make migrate`: Run database migrations.
*   `make seed`: Seed initial data.
*   `make clean`: Clean build artifacts.
*   `make rebuild`: Re-build all modules.
*   `make rebuild-frontened`: Re-build the frontend.
*   `make rebuild-backend`: Re-build the backend.

### Frontend Development

The frontend is an Angular application located in the `frontend` directory.

*   `npm install`: Install dependencies.
*   `npm start`: Start the development server.
*   `npm run build`: Build for production.
*   `npm test`: Run tests.
*   `npm run lint`: Run linting.

## Development Conventions

*   The backend code follows the standard Java and Spring Boot conventions.
*   The frontend code follows the standard Angular conventions.
*   The project uses JaCoCo for code coverage.
*   The project uses Spotless for code formatting.
*   The project uses OWASP Dependency-Check for security vulnerability scanning.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/balemneh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
