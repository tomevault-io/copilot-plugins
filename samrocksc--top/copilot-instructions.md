## top

> This document provides essential information for agents working with this codebase.

# AGENTS.md

This document provides essential information for agents working with this codebase.

## Project Overview

This is a full-stack TypeScript application with:
- **Backend**: Express.js API with TypeScript, Prisma ORM, and PostgreSQL
- **Frontend**: Next.js 13+ with TypeScript and Tailwind CSS
- **Database**: PostgreSQL via Prisma
- **Authentication**: Auth0 with JWT tokens
- **Infrastructure**: Docker Compose for local development services (PostgreSQL, Redis)

## Project Structure

```
.
├── backend/          # Express TypeScript API
│   ├── src/          # Source code
│   │   ├── db/       # Database layer with Prisma
│   │   ├── config/   # Configuration files (OpenAPI)
│   │   └── server.ts   # Main server file
│   ├── prisma/       # Prisma schema and migrations
│   ├── scripts/      # Utility scripts
│   └── package.json  # Backend dependencies
├── client/           # Client frontend (Next.js)
├── docker-compose.yml # Docker services configuration
└── Makefile          # Build and deployment commands
```

## Essential Commands

### Setup and Development
```bash
# Install all dependencies
make install-deps

# Set up the entire application
make setup

# Start all services (PostgreSQL, Redis)
make start

# Run database migrations
make migrate-dev

# Set up client environment with Auth0 configuration
make client-env-setup

# Start backend in development mode
make backend-dev

# Start client frontend (also starts backend in background)
make client-dev

# Start both backend and frontend in development mode
make dev
```

### Code Generation
```bash
# Generate Prisma client
make generate

# Generate OpenAPI specification
make generate-openapi

# Generate frontend TypeScript types from OpenAPI spec
make generate-frontend-types
```

### Database Management
```bash
# Reset the database
make reset-db

# Run production database migrations
make migrate-deploy
```

### Maintenance
```bash
# View logs for all services
make logs

# Stop all services
make stop

# Clean up (stop and remove containers)
make clean

# Rebuild everything from scratch
make rebuild
```

### Backend Development
```bash
# To start the backend server, use:
npm run dev
```

## Backend API

The backend is an Express.js server running on port 8000 with:
- Prisma ORM for database operations
- Auth0 authentication with JWT bearer tokens
- OpenAPI/Swagger documentation
- CORS and Helmet security middleware

### Key Endpoints
- `GET /` - Health check endpoint
- `GET /health` - Health check with database connectivity
- `GET /user/:auth0Id` - Get user by Auth0 ID
- `GET /api-docs` - Swagger UI documentation
- `GET /openapi.json` - Raw OpenAPI specification

### Authentication
All endpoints (except health checks) require a valid Auth0 JWT token in the Authorization header:
```
Authorization: Bearer <token>
```

## Frontend (Client)

The frontend is a Next.js 13+ application using the app directory structure:
- Running on port 3001 in development
- Uses React Server Components by default
- Auth0 authentication via @auth0/nextjs-auth0
- Tailwind CSS for styling

### Key Pages
- `/` - Home page
- `/auth/login` - Login page
- `/auth/callback` - Auth0 callback handler
- `/lists` - Lists overview
- `/lists/new` - Create new list

## Database Schema

The application uses PostgreSQL with Prisma ORM. Current schema:

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  auth0Id   String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## Code Patterns and Conventions

### Backend
1. **Route Documentation**: All routes are documented with OpenAPI JSDoc annotations
2. **Error Handling**: Async route handlers should use try/catch blocks
3. **Database Operations**: Use Prisma client through userService functions
4. **Environment Variables**: Access via `process.env` with dotenv loading
5. **Authentication**: Middleware handles JWT validation automatically

### Frontend
1. **Components**: Use TypeScript with React functional components
2. **Styling**: Tailwind CSS classes preferred over custom CSS
3. **API Calls**: Use generated TypeScript types from OpenAPI spec
4. **Authentication**: Use @auth0/nextjs-auth0 for session management

## Important Gotchas

1. **Port Conflicts**: Backend runs on 8000, frontend on 3001
2. **Database Dependencies**: Ensure PostgreSQL is running via Docker before starting the backend
3. **Auth0 Setup**: Requires manual configuration of .env.local in client directory
4. **Type Generation**: Frontend types depend on backend being accessible at http://localhost:3000/api-docs/openapi.json
5. **Migration Workflow**: Database changes require creating new migrations with `prisma migrate dev`
6. **Secret Management**: Never commit actual secrets; use `.env.example` as a template

## Testing and Debugging

### Backend Debugging
1. Check logs with `make logs`
2. Verify database connectivity with the `/health` endpoint
3. Test individual routes with tools like curl or Postman (include JWT headers)

### Frontend Debugging
1. Check browser console for errors
2. Verify environment variables are properly set in `.env.local`
3. Check network tab for failed API requests

## Adding New Features

### Backend API Endpoints
1. Add route with JSDoc OpenAPI annotations
2. Implement route handler in `src/server.ts`
3. Add service functions in `src/db/` as needed
4. Run `make generate-openapi` to update specification
5. Run `make generate-frontend-types` to update frontend types

### Database Changes
1. Update `prisma/schema.prisma`
2. Run `make generate` to update Prisma client
3. Create migration with `make migrate-dev`
4. Update service functions in `src/db/` as needed

### Frontend Features
1. Create new pages in `client/src/app/`
2. Add new components in `client/src/components/`
3. Use generated types for API responses
4. Follow existing styling patterns with Tailwind CSS

## Development Workflow

1. Start services: `make start`
2. Run database migrations: `make migrate-dev`
3. Generate Prisma client: `make generate`
4. Set up client environment: `make client-env-setup`
5. Start development servers: `make dev`
6. Make code changes
7. Generate new types when backend API changes: `make generate-frontend-types`

## Secret Management

This repository has several measures in place to prevent secrets from being committed:
1. **.gitignore**: Ignores common secret files like `.env`, `*.key`, etc.
2. **Pre-commit Hook**: Warns about potential secrets in staged files
3. **Setup Script**: Provides `scripts/setup-hooks.sh` to install the pre-commit hook
4. **Best Practices**: Uses environment variables for configuration with `.env.example` templates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samrocksc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samrocksc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
