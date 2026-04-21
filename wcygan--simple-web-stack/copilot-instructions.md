## simple-web-stack

> This document outlines the requirements for a simple todo list application designed to help users manage their tasks effectively. The application will feature a frontend built with Deno Fresh for a fast and modern user interface, enabling users to view, add, edit, mark as complete, and delete tasks. The backend, powered by Axum and Rust, will provide a robust and efficient API for managing task data, including CRUD (Create, Read, Update, Delete) operations. All task information will be persistently stored in a MySQL database, with SQLx utilized for asynchronous, type-safe database interactions, ensuring data integrity and reliable task management. To ensure reliability and correctness, the application will undergo extensive unit and integration testing, leveraging Testcontainers (through testcontainers-rs) to manage and isolate external dependencies like the MySQL database during the testing phase, facilitating a consistent and dependable testing environment.

# Simple Web Stack Architecture Guide

## Application Overview

This document outlines the requirements for a simple todo list application designed to help users manage their tasks effectively. The application will feature a frontend built with Deno Fresh for a fast and modern user interface, enabling users to view, add, edit, mark as complete, and delete tasks. The backend, powered by Axum and Rust, will provide a robust and efficient API for managing task data, including CRUD (Create, Read, Update, Delete) operations. All task information will be persistently stored in a MySQL database, with SQLx utilized for asynchronous, type-safe database interactions, ensuring data integrity and reliable task management. To ensure reliability and correctness, the application will undergo extensive unit and integration testing, leveraging Testcontainers (through testcontainers-rs) to manage and isolate external dependencies like the MySQL database during the testing phase, facilitating a consistent and dependable testing environment.

## Tech Stack Overview

This is a full-stack web application with clear separation between frontend, backend, and infrastructure:

- **Frontend**: Deno Fresh v2.0 + Preact + Tailwind CSS (Port 8000)
- **Backend**: Rust Axum + Tokio async runtime (Port 3000)
- **Database**: MySQL + sqlx
- **Infrastructure**: Docker Compose + GitHub Actions CI/CD

## Project Structure

### Frontend ([frontend/](mdc:frontend))
- **Entry Point**: [main.ts](mdc:frontend/main.ts) - Fresh app initialization
- **Config**: [deno.json](mdc:frontend/deno.json) - Fresh configuration and dependencies

### Backend ([backend/](mdc:backend))
- **Entry Point**: [main.rs](mdc:backend/src/main.rs) - Server startup and configuration
- **Dependencies**: [Cargo.toml](mdc:backend/Cargo.toml) - Rust dependencies and project metadata

### Infrastructure
- **Docker**: [docker-compose.yml](mdc:docker-compose.yml) - Service orchestration
- **Tasks**: [deno.json](mdc:deno.json) - Project-level tasks and scripts
- **CI/CD**: [scripts/run-ci.ts](mdc:scripts/run-ci.ts) - Local CI testing with act

## Development Patterns

### Backend Patterns
- **Async-first**: All handlers use async/await with Tokio runtime
- **JSON APIs**: Structured responses using serde serialization
- **Error Handling**: Proper HTTP status codes and error responses
- **Testing**: Unit tests embedded in source + integration tests with TestContainers-rs
- **Instrumentation**: Structured logging with tracing crate

### Frontend Patterns
- **Islands Architecture**: Interactive components in [islands/](mdc:frontend/islands) with server-side rendering
- **Signals**: State management using Preact signals for reactivity
- **File-based Routing**: Routes automatically generated from [routes/](mdc:frontend/routes) directory
- **API Integration**: Backend communication through [routes/api/](mdc:frontend/routes/api) handlers

### Development Workflow
- **Local Development**: `deno task up` - Starts all services with live reloading
- **Testing**: `deno task test:backend` - Runs comprehensive test suite
- **CI/CD**: `deno task ci` - Local GitHub Actions testing with act

## Key Integration Points

1. **Frontend ↔ Backend**: HTTP API calls from Fresh routes/islands to Axum endpoints
2. **Backend ↔ Database**: sqlx for async MySQL operations
3. **Docker Services**: Containerized services with development volumes
4. **Data Flow**: Frontend signals → API calls → Database operations → JSON responses → UI updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wcygan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
