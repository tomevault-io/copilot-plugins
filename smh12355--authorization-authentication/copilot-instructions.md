## authorization-authentication

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ASP.NET Core 10.0 Web API with Keycloak authentication, PostgreSQL database, and MinIO S3-compatible storage. Uses feature-based architecture with infrastructure abstractions.

## Common Commands

### Infrastructure (Docker)
```bash
cd Docker
docker-compose up -d              # Start all services (Keycloak, PostgreSQL x2, MinIO)
docker-compose down               # Stop services
docker-compose down -v            # Stop and remove volumes (deletes data)
docker-compose ps                 # Check service status
docker-compose logs keycloak      # View Keycloak logs
```

### Application
```bash
cd Src/Authorization~authentication
dotnet restore                    # Restore dependencies
dotnet build                      # Build project
dotnet run                        # Run API (http://localhost:5275)
```

### Database Migrations (EF Core)
```bash
cd Src/Authorization~authentication
dotnet tool update --global dotnet-ef    # Update EF Core CLI tool
dotnet ef migrations add MigrationName   # Create new migration
dotnet ef database update                # Apply pending migrations
dotnet ef database update MigrationName  # Rollback to specific migration
dotnet ef database drop                  # Drop entire database
```

## Architecture

### Service Registration Pattern
All infrastructure and feature services are registered via extension methods in Program.cs:
- `AddKeycloakAuthentication()` - JWT Bearer auth with Keycloak
- `AddMinIO()` - S3-compatible file storage
- `AddDbContext()` - PostgreSQL with EF Core
- `RegisterFeaturesServices()` - Feature-specific services (e.g., IFileStorageService)
- `AddOwnApiVersioning()` - URL segment versioning (e.g., /api/v1/, /api/v2/)
- `AddOwnOpenApi()` - OpenAPI/Swagger configuration

### Feature-Based Structure
Features are organized in `Features/` directory, each containing:
- Controllers (API endpoints)
- Models (DTOs, requests, responses)
- Services (business logic)

Example: `Features/FileUpload/` contains FileUploadController, models, and MinioFileStorageService.

### Infrastructure Layer
Cross-cutting concerns in `Infrastructure/`:
- **Database**: AppDbContext, entities, EF Core configuration
- **Keycloak**: JWT authentication setup, token validation
- **MinIO**: S3 client configuration, health checks, bucket initialization
- **OpenApi**: Swagger/Scalar API documentation

### Configuration Notes
- **Two PostgreSQL instances**:
  - Keycloak DB on port 5432 (internal to Docker)
  - Main app DB on port 6432 (mapped from container)
- **Connection string**: `appsettings.Development.json` → `ConnectionStrings:MainDb`
- **Auth config mismatch**: `appsettings.json` uses `Keycloak` section, `appsettings.Development.json` uses `AuthOptions` section with different realm names
- **MinIO**: Accessible at http://localhost:9000 (API) and http://localhost:9001 (Console)

### C# 12 Extension Methods
Project uses C# 12 extension syntax:
```csharp
extension(IServiceCollection services)
{
    public IServiceCollection AddSomething() { ... }
}
```

### API Versioning
Controllers use URL segment versioning via `[ApiVersion("1.0")]` attribute. Versions appear in URL as `/api/v1/endpoint`.

### Global Exception Handling
Custom middleware in `Middleware/ExceptionHandlingMiddleware.cs` registered via `UseGlobalExceptionHandling()` extension.

## Keycloak Setup
1. Access admin console: http://localhost:8080/admin (admin/admin)
2. Create realm matching config (check `appsettings.Development.json` for realm name)
3. Create client matching `ClientId` in config
4. Create roles and users as needed
5. Get client secret from Keycloak and update `ClientSecret` in config

## Health Checks
Available at `/health` endpoint. Monitors:
- Main PostgreSQL database
- MinIO storage

## Development Notes
- No test projects currently exist
- Swagger UI available at root URL (/) in Development mode
- HTTPS redirection enabled
- Sensitive data logging enabled in Development environment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smh12355) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
