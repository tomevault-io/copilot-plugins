## fvmms-backend

> This document provides a comprehensive overview of the `QuanLyTauCa_DACN/Server` project to give context for future AI-assisted development sessions.

# GEMINI.md: Project Context
This document provides a comprehensive overview of the `QuanLyTauCa_DACN/Server` project to give context for future AI-assisted development sessions.
## Project Overview
This is the backend server for a **Fishing Vessel Monitoring and Management System (FVMMS)**. It is a Node.js application written in TypeScript, using the Express.js framework. The server exposes a REST API to be consumed by a front-end application (e.g., Flutter/WPF).
### Core Technologies
*   **Runtime**: Node.js
*   **Framework**: Express.js
*   **Language**: TypeScript
*   **Database**: PostgreSQL
*   **ORM**: Prisma
*   **Authentication**: JWT (JSON Web Tokens) with role-based access control.
*   **Validation**: Joi for request body validation.
*   **API Documentation**: Swagger (OpenAPI) via `swagger-jsdoc` and `swagger-ui-express`.
*   **Password Hashing**: `bcrypt`

### Architecture

The project follows a **feature-based architecture**. Core business logic is organized into feature modules within the `src/features` directory (e.g., `src/features/auth`).

Shared and cross-cutting concerns are handled in dedicated directories:
*   `src/config`: For application and database configuration.
*   `src/middlewares`: For shared middleware like error handling and authentication checks.
*   `src/utils`: For common utilities like password hashing, JWT management, and logging.

The database schema is managed declaratively with Prisma in `prisma/schema.prisma`, which defines a comprehensive data model including users, vessels, trips, compliance, and more.

## Building and Running

Key scripts are defined in `package.json`.

### Prerequisites

*   Node.js
*   A running PostgreSQL database.
*   An `.env` file created from the `.env.example` template with the correct database connection string and JWT secret.

### Key Commands

*   **Install dependencies**:
    ```bash
    npm install
    ```

*   **Run in development mode** (with hot-reloading):
    ```bash
    npm run dev
    ```

*   **Build for production** (compiles TypeScript to JavaScript in the `dist` folder):
    ```bash
    npm run build
    ```

*   **Run in production**:
    ```bash
    npm start
    ```

*   **Prisma commands**:
    *   Generate Prisma Client:
        ```bash
        npx prisma generate
        ```
    *   Apply database migrations:
        ```bash
        npx prisma migrate dev
        ```

### API Documentation

Once the server is running, the Swagger UI for API documentation is available at `http://localhost:<PORT>/api-docs` (e.g., http://localhost:4000/api-docs).

## Development Conventions

*   **Structure**: Code is organized by feature. Each feature folder contains its own routes, controllers, services, and validation schemas.
*   **Validation**: All incoming request bodies are validated using Joi schemas defined in `*.validation.ts` files.
*   **Authentication**: Routes are protected using the `authenticate` and `authorize` middlewares from `src/middlewares/auth.ts`.
*   **Error Handling**: A global error handler (`src/middlewares/error.ts`) catches and formats all unhandled errors.
*   **Database Schema**: Any changes to the database model should be made in `prisma/schema.prisma` and applied via a new migration.
*   **Environment**: All secrets and environment-specific configurations (database URL, JWT secret, port) must be managed in an `.env` file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TruongHieuHuy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
