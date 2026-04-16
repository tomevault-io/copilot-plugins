## bankaccount-travis

> **Name:** bankaccount-travis

# Bank Account Travis - Project Documentation

## Project Overview

**Name:** bankaccount-travis

**Description:** Bank Account Management System - Java Spring Boot application with Maven build, Travis CI integration, and comprehensive test automation. Demonstrates TDD and object-oriented design principles.

## Requirements

- Java 8+ with Spring Boot framework
- Maven for dependency management and build
- JUnit for unit testing
- In-memory repository for account storage
- REST API endpoints for account operations

## Project Structure

### Source Code Organization

- **Source Root:** `src/main/java`
- **Test Root:** `src/test/java`
- **Build System:** Maven
- **Entry Point:** `src/main/java/com/danielwildt/api/WildtechBank.java`

### Directory Structure

| Directory | Description |
|-----------|-------------|
| `src/main/java/com/danielwildt/` | Main source code directory |
| `src/main/java/com/danielwildt/api/` | REST API controllers and main application |
| `src/main/java/com/danielwildt/model/` | Domain models (BankAccount) |
| `src/main/java/com/danielwildt/db/` | Repository pattern implementation |
| `src/main/java/com/danielwildt/console/` | Console application interface |
| `src/test/java/` | Unit tests following same package structure |
| `pom.xml` | Maven project configuration |
| `build.xml` | Ant build configuration (alternative) |

## Key Features

- Bank account creation and management
- Balance checking and withdrawal operations
- In-memory account repository pattern
- REST API with Spring Boot
- Comprehensive unit test coverage
- Travis CI continuous integration
- Heroku deployment support

## Technology Stack

- **Language:** Java
- **Framework:** Spring Boot 2.1.0
- **Test Framework:** JUnit 4.12
- **Build Tool:** Maven 3.x
- **Java Version:** 1.8

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/balance?name={accountName}` | Check account balance |
| POST | `/add?name={accountName}&balance={amount}` | Create account |
| POST | `/withdraw?name={accountName}&value={amount}` | Withdraw from account |

## Code Conventions

### Naming Conventions

- **Package Naming:** `com.danielwildt.{module}`
- **Class Naming:** PascalCase for classes, camelCase for methods
- **Test Naming:** TestClass naming convention with descriptive method names

### Architecture Patterns

- **Repository Pattern:** Interface + InMemory implementation
- **TDD Approach:** Test-driven development methodology
- **Spring Boot:** Convention over configuration principles

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dwildt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
