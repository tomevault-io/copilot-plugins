## nestjs-microservices

> Global rules for microservices project


# NestJS Microservices Architecture

## Package Management
- Use pnpm for all package installations: `pnpm add <package>` 
- Avoid using npm or yarn commands
- Use workspace protocol for internal dependencies: `pnpm add @project/<package-name>`
- Run commands in specific workspaces with `pnpm --filter <workspace> <command>`

## Monorepo Structure
- This project uses pnpm workspaces with Turborepo
- Service packages are in `/services` directory
- Shared packages are in `/packages` directory
- Follow package isolation - only import shared code from shared packages

## Code Organization
- Each service follows NestJS module-based architecture
- Keep controllers thin - business logic belongs in services
- Follow NestJS naming conventions (*.controller.ts, *.service.ts, *.module.ts)
- Use DTOs for data validation and transformation
- Place database schemas in the `/db` directory of each service
- Authentication logic is in the `/auth` directory

## Authentication & Authorization
- This project uses Keycloak for identity management
- JWT authentication is implemented with `@nest-shared/jwt-auth`
- Use decorators from `/decorators` for authorization
- Protect routes with appropriate guards from `/guards`

## Database
- Drizzle ORM is used for database operations
- Schema definitions are in `src/db/schema.ts` of each service
- Run migrations with `drizzle-kit`
- Database connections are configured in `app-config`

## Docker & Deployment
- Use docker-compose.dev.yml for local development
- Use docker-compose.yml for production deployment
- Services are containerized using Dockerfiles in `/docker` directory
- Environment variables are stored in .env files (use env.example as template)

## API Gateway
- All external requests go through the API Gateway service
- The API Gateway handles routing to appropriate microservices
- Authentication is validated at the gateway level
- Use appropriate decorators for public endpoints


## Naming Conventions
- Use kebab-case for file names
- Use PascalCase for classes and interfaces
- Use camelCase for variables, functions, and methods
- Use ALL_CAPS for constants and environment variables

## Best Practices
- Follow the Single Responsibility Principle
- Use dependency injection for all services
- Handle errors consistently across all services
- Document all public APIs
- Use strong typing, avoid `any` type

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lequochung19971)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lequochung19971)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
