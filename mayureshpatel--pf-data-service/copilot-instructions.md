## pf-data-service

> This document outlines the specific rules, commands, and structures for the Spring Boot backend service. It acts as an extension to the root `GEMINI.md` file.

# pf-data-service Instructions

This document outlines the specific rules, commands, and structures for the Spring Boot backend service. It acts as an extension to the root `GEMINI.md` file.

## Build and Run Commands
When modifying or interacting with the backend, utilize the following Maven commands:
- **Build the project:** `mvn clean install`
- **Run standard tests:** `mvn test`

## Package Structure
The project is a single-module application located under `src/main/java/com/mayureshpatel/pfdataservice`. Enforce the following package organization:

- **`aspect`**: Aspect-oriented programming logic.
- **`config`**: Application configuration files and Spring `@Configuration` classes.
- **`controller`**: REST API endpoints (following kebab-case plural nouns).
- **`domain`**: Entities that strictly mirror database tables.
- **`dto`**: Data Transfer Objects passed to and from the UI.
- **`exception`**: Custom exceptions and `@ExceptionHandler` controllers.
- **`filter`**: Interceptors and servlet filters.
- **`mapper`**: Manual mapping logic between DTOs and Domain objects (Reminder: Do not use mapper libraries).
- **`repository`**: Data access layer exclusively utilizing Spring JDBC Client.
- **`security`**: Spring Security configurations.
- **`service`**: Core business logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mayureshpatel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mayureshpatel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
