## refnotes

> This is a note-taking application called **RefNotes**. It's a full-stack web application with a .NET backend and an Angular frontend.

# RefNotes Project

## Project Overview

This is a note-taking application called **RefNotes**. It's a full-stack web application with a .NET backend and an Angular frontend.

- **Backend:** The backend is built with .NET, ASP.NET Core, and Entity Framework Core. It uses a MySQL database for data storage and Redis for caching. The backend API is located in the `src/Api` directory.
- **Frontend:** The frontend is an Angular single-page application (SPA) located in the `src/Web` directory. It uses Tailwind CSS for styling.
- **Containerization:** The project is fully containerized using Docker and Docker Compose. The `compose.yaml` file defines the services for the backend, frontend, database, and a Caddy reverse proxy.

## Src Directory Structure

The `src` directory contains the source code for the application, organized as follows:

- **AppHost:** A .NET Aspire project that orchestrates the different parts of the application.
- **Api:** The backend REST API, built with ASP.NET Core.
- **Web:** The Angular SPA frontend.
- **Data:** Contains the database context and migrations.
- **ServiceDefaults:** A project with default service configurations.
- **MailDev.Hosting:** A project for hosting a local SMTP server for development.
- **MigrationService:** A service for running database migrations.

## Building and Running

The easiest way to run the project is with .NET Aspire. You'll need to have the .NET SDK, Node.js, and pnpm installed.

1.  **Start the backend and frontend:**
    ```bash
    cd src/AppHost
    dotnet run
    ```

The application will be available at `http://localhost:4200`.

## Testing

### Backend Tests

To run the backend tests, navigate to the `test` directory and run:

```bash
dotnet test
```

### Frontend Tests

To run the frontend tests, navigate to the `src/Web` directory and run:

```bash
pnpm test
```

## Development Conventions

- **Backend:** The backend follows standard .NET conventions. It uses a service-oriented architecture with controllers, services, and repositories.
- **Frontend:** The frontend follows standard Angular conventions. It uses a component-based architecture with services for data access.
- **Code Style:** The project uses `.editorconfig` to enforce consistent code style.
- **Git:** The project uses Git for version control. Please follow standard Git best practices when contributing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LukaStrbad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LukaStrbad)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
