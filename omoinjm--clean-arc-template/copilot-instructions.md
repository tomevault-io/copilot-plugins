## clean-arc-template

> This document provides a comprehensive overview of the Clean Architecture Template project, designed to help Gemini and other developers understand its structure, purpose, and key components.

# Gemini Code Assistant Report

This document provides a comprehensive overview of the Clean Architecture Template project, designed to help Gemini and other developers understand its structure, purpose, and key components.

## Project Overview

This project is a .NET solution template for creating REST APIs with ASP.NET Core, following the principles of Clean Architecture. The template is designed to provide a straightforward and efficient approach to enterprise application development, with a clear separation of concerns.

The architecture is divided into four main layers:

-   **Core:** Contains domain entities, business logic, repository interfaces, and service interfaces.
-   **Application:** Implements the CQRS pattern with commands, queries, and handlers using MediatR. It also includes DTOs, response objects, and AutoMapper profiles.
-   **Infrastructure:** Provides the implementation for repositories, external service integrations, caching, and database queries.
-   **API:** The presentation layer, which is an ASP.NET Core Web API with controllers, dependency injection configuration, and middleware setup.

## Technology Stack

-   **Framework:** ASP.NET Core 9.0
-   **Language:** C#
-   **Pattern:** Clean Architecture, CQRS
-   **Libraries:**
    -   MediatR: For CQRS implementation
    -   AutoMapper: For object mapping
    -   Dapper: For data access
    -   Npgsql: For PostgreSQL database access
    -   Swagger/Swashbuckle: For API documentation

## Building and Running

### Build the Solution

```bash
dotnet build
```

### Run the API

```bash
dotnet run --project src/Clean.Architecture.Template.API
```

The API will be available at `https://localhost:5001` by default.

### Run Tests

```bash
dotnet test
```

## Development Conventions

### Command and Query Separation (CQRS)

-   Use **Commands** for operations that modify state (e.g., Create, Update, Delete).
-   Use **Queries** for operations that retrieve data (e.g., Get, List).
-   Implement corresponding handlers in the `Application/Handlers` folder.

### Dependency Injection

-   Register services in `src/Clean.Architecture.Template.API/Startup.cs`.
-   Use constructor injection throughout the application.

### Entity and DTO Separation

-   Domain entities are kept clean with no framework dependencies.
-   Response DTOs are used to shape API responses.
-   AutoMapper is used to map entities to DTOs.

### Repository Pattern

-   Repository interfaces are defined in the `Core` layer.
-   Repository implementations are in the `Infrastructure` layer.

## Key Files

-   `Clean.Architecture.Template.sln`: The solution file that defines the project structure.
-   `src/Clean.Architecture.Template.API/Startup.cs`: The entry point for the API, where services are configured and the application is set up.
-   `src/Clean.Architecture.Template.API/Controllers/`: Contains the API controllers that handle HTTP requests.
-   `src/Clean.Architecture.Template.Application/Handlers/`: Contains the MediatR handlers for commands and queries.
-   `src/Clean.Architecture.Template.Core/Entity/`: Contains the domain entities.
-   `src/Clean.Architecture.Template.Infrastructure/Repository/`: Contains the repository implementations for data access.
-   `docs/`: Contains detailed documentation for the project, including a quick start guide, API documentation, and a testing guide.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omoinjm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/omoinjm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
