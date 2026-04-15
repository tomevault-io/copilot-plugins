## monorepo

> `gnas-starter` is a Spring Boot 3 (Java 17) based microservices monorepo designed for a financial system. It features multiple services focused on banking operations like account management, transactions, and loans, tied together by shared logic and a common build system.

# GEMINI.md - gnas-starter Context

## Project Overview
`gnas-starter` is a Spring Boot 3 (Java 17) based microservices monorepo designed for a financial system. It features multiple services focused on banking operations like account management, transactions, and loans, tied together by shared logic and a common build system.

### Core Architecture
- **Monorepo Structure**: Managed by Gradle with shared build logic.
- **Microservices**:
  - `account-service`: Manages bank accounts and balances.
  - `transaction-service`: Handles financial transactions (via REST/Kafka).
  - `loan-service`: Manages loan applications and processing.
  - `notification-service`: Sends notifications based on system events.
  - `audit-service`: Records audit logs for system activities.
- **Shared Components**:
  - `shared-lib`: Common utilities, models, and shared library logic.
  - `build-logic`: Gradle convention plugins (`SpringBootMicroserviceConventionPlugin`, `SpringBootSharedLibraryConventionPlugin`) for standardized builds across modules.
- **Infrastructure**: Uses PostgreSQL for persistence and Kafka for event-driven messaging.

### Main Technologies
- **Language**: Java 17
- **Framework**: Spring Boot 3.3.4
- **Build System**: Gradle 8.x
- **Persistence**: PostgreSQL, Hibernate/JPA, Flyway for migrations.
- **Messaging**: Apache Kafka (Spring Kafka).
- **Containerization**: Docker Compose for local infrastructure.

---

## Building and Running

### Prerequisites
- JDK 17+
- Docker & Docker Compose (for infrastructure)

### Key Commands

- **Build everything**:
  ```bash
  ./gradlew build
  ```

- **Run a specific service**:
  ```bash
  ./gradlew :account-service:bootRun
  ```

- **Run infrastructure (Postgres, Kafka)**:
  ```bash
  docker compose -f docker/docker-compose.yaml up -d
  # Some services might have specific docker-compose files (e.g., account-service/docker/docker-kafka-compose.yaml)
  ```

- **Clean build**:
  ```bash
  ./gradlew clean build
  ```

- **Publish shared-lib to local Maven repository**:
  ```bash
  ./gradlew :shared-lib:publishToMavenLocal
  ```

---

## Development Conventions

### Build Logic & Plugins
- **Convention Plugins**: Use `com.gnas.starter.spring-boot-microservice` for new microservices and `com.gnas.starter.spring-boot-shared-lib` for shared libraries. These are defined in `build-logic`.
- **Java Version**: Fixed to Java 17 in convention plugins.
- **Dependencies**: Spring Boot BOM (version 3.3.4) is used to align dependency versions.

### Coding Style
- **Lombok**: Extensively used for boilerplate reduction (Getters, Setters, Constructors).
- **RESTful APIs**: Standard Spring MVC controllers. `account-service` uses `/account-service` as its context path.
- **Database Migrations**: Flyway is used for schema management. Migrations are located in `src/main/resources/db/migration`.
- **Event-Driven**: Prefers Kafka for asynchronous communication (e.g., Notifications, Audits).

### Testing
- **Framework**: JUnit 5 with Spring Boot Test.
- **Execution**:
  ```bash
  ./gradlew test
  ```
- **Convention**: Convention plugins automatically include `spring-boot-starter-test` and configure the JUnit Platform.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nguyensang521290)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nguyensang521290)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
