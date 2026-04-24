## evachild-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Start development server**: `pnpm run dev` (uses ts-node-dev with hot reload)
- **Run database migrations**: `pnpm run migrate` (equivalent to `npx prisma migrate dev`)
- **Package manager**: Uses pnpm (specified in packageManager field)

## Architecture Overview

EvaChild API is a Node.js/Express backend using Prisma ORM for MySQL database operations. The codebase follows a clean layered architecture pattern:

### Directory Structure
```
src/
├── controllers/     # Express route handlers - minimal logic, delegate to services
├── services/        # Business logic and Prisma database queries
├── routes/          # Route definitions, one file per domain model
├── middleware/      # Authentication and other middleware
├── prisma/          # Prisma client configuration
└── server.ts        # Main Express app, registers routes under /api/*
```

### Key Patterns
- **Service Layer**: All database access goes through service files. Controllers never query Prisma directly.
- **RESTful API**: Endpoints grouped by resource (`/api/users`, `/api/roles`, `/api/classes`, `/api/children`, `/api/auth`)
- **Standard CRUD**: Each resource supports GET (all/by id), POST (create), PUT (update), DELETE operations
- **Error Handling**: Controllers catch errors and respond with status 500 and `{ error: "Internal Server Error" }`

## Database Schema

Uses MySQL with Prisma ORM. Key models:
- **User**: Users with roles (role-based access)
- **Role**: User roles and permissions
- **Child**: Children enrolled in classes
- **Class**: Age-based class groupings
- **Milestone**: Development milestones tracked per child
- **Evaluation**: Child assessments and scoring

Database connection configured via `DATABASE_URL` environment variable.

## File Naming Conventions

- **Services/Controllers**: Singular names (e.g., `userService.ts`, `childController.ts`)
- **Routes**: Plural names (e.g., `userRoutes.ts`, `childRoutes.ts`)
- **Models**: Defined in `prisma/schema.prisma` using PascalCase

## TypeScript Configuration

- Uses ES modules (`"type": "module"` in package.json)
- Allows TypeScript imports with `.ts` extensions
- Strict mode enabled with additional type safety options
- Path mapping: `@/*` points to `src/*`

## Adding New Resources

1. Define model in `prisma/schema.prisma`
2. Run `pnpm run migrate` to apply schema changes
3. Create service file in `src/services/`
4. Create controller file in `src/controllers/`
5. Create routes file in `src/routes/`
6. Register routes in `src/server.ts`

## Authentication

Uses JWT-based authentication with bcrypt for password hashing. Auth middleware and routes are implemented for user registration and login flows.

## Git Commit Guidelines

- NEVER include Claude Code promotional text or links in commit messages
- Keep commit messages clear, concise, and professional
- Follow conventional commit format: `type: description`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fedeparrilla10) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
