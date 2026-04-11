## authzsvc-full

> This document provides a comprehensive overview of the Go-based authentication and authorization service, as understood and analyzed by Gemini.

# Gemini Project Overview: Auth + RBAC + SMS OTP (Go)

This document provides a comprehensive overview of the Go-based authentication and authorization service, as understood and analyzed by Gemini.

## Project Description

This project is a Go-based microservice that provides authentication, authorization (RBAC), and SMS-based One-Time Password (OTP) functionalities. It is built following the principles of Clean Architecture and Hexagonal Architecture, ensuring a clear separation of concerns and a high degree of maintainability.

## How to Run the Project

To run the project, you need to have Docker and Docker Compose installed.

1.  **Create the environment file:**
    ```bash
    cp .env.example .env
    ```

2.  **Build and run the services:**
    ```bash
    docker compose up --build
    ```

## API Endpoints

The service exposes the following RESTful API endpoints:

*   **Authentication:**
    *   `POST /auth/register`: Register a new user.
    *   `POST /auth/otp/verify`: Verify the SMS OTP.
    *   `POST /auth/login`: Log in a user.
    *   `POST /auth/refresh`: Refresh an expired access token.
    *   `POST /auth/logout`: Log out a user.
    *   `GET /auth/me`: Get the details of the currently logged-in user.

*   **Administration:**
    *   `GET /admin/policies`: Get all authorization policies.
    *   `POST /admin/policies`: Create a new authorization policy.
    *   `DELETE /admin/policies`: Delete an authorization policy.

## Architecture

The application follows the principles of **Clean Architecture** and **Hexagonal Architecture**. This results in a clear separation of concerns between the different layers of the application.

*   **Domain Layer (`/domain`):** Contains the core business logic, entities, and interfaces of the application. It is completely independent of any external frameworks or libraries.

*   **Application Layer (`/internal/services`):** Orchestrates the business logic defined in the domain layer. It is responsible for handling use cases and coordinating the different domain services.

*   **Infrastructure Layer (`/internal/infra`):** Contains the implementation of the interfaces defined in the domain layer. This includes database repositories, caching services, and other external service integrations.

*   **Presentation Layer (`/internal/http` and `/cmd/authzsvc`):** The entry point of the application. It is responsible for handling HTTP requests, routing, and middleware.

## Key Architectural Patterns

The project leverages several key architectural patterns to ensure a high degree of quality and maintainability:

*   **Repository Pattern:** Abstracts the data access logic, allowing the business logic to be independent of the underlying data store.
*   **Factory Pattern:** Used for creating instances of services and other dependencies, facilitating dependency injection.
*   **Chain of Responsibility:** Implemented as a middleware pipeline for handling HTTP requests, allowing for cross-cutting concerns like authentication and authorization to be handled in a modular way.

## Security

The service implements a robust security model with the following features:

*   **JWT Tokens:** Uses JSON Web Tokens (JWT) for authentication, with a refresh token mechanism.
*   **Session Management:** Manages user sessions using Redis for efficient and scalable session storage.
*   **Password Security:** Uses bcrypt for hashing passwords, with a configurable strength.
*   **RBAC:** Implements Role-Based Access Control (RBAC) using Casbin for fine-grained authorization.
*   **OTP Verification:** Provides SMS-based OTP verification for an extra layer of security.

## Testing

The project has a comprehensive testing framework with the following features:

*   **Table-Driven Tests:** Uses table-driven tests to ensure that all business logic paths are tested.
*   **Mocking:** Mocks all external dependencies to ensure that tests are isolated and run quickly.
*   **High Test Coverage:** Aims for a high test coverage of over 95% for critical components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andremarcelonunes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andremarcelonunes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
