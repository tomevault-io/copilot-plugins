## vacation-requests

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vacation Request Application built with:
- **Next.js 15.3.0** (App Router) with TypeScript
- **Prisma ORM** with PostgreSQL
- **NextAuth.js v5** for authentication (Microsoft Entra ID)
- **shadcn/ui** components with Tailwind CSS v4
- **Vitest** and **Playwright** for testing

## Essential Commands

```bash
# Development
npm run dev              # Start development server with Turbopack

# Building and Production
npm run build           # Build for production (runs prisma generate first)
npm start              # Start production server

# Testing
npm test               # Run unit tests
npm run test:ui        # Run tests with Vitest UI
npm run coverage       # Generate coverage report
npm run test:e2e       # Run Playwright E2E tests
npm run test:e2e:ui    # Run Playwright with UI

# Database
npm run migrate:dev    # Run Prisma migrations in development
npm run db:reset       # Reset database and re-seed
npm run prisma:seed    # Seed database with initial data

# Code Quality
npm run lint           # Run ESLint
```

## High-Level Architecture

### Authentication Flow
- Server-side authentication via NextAuth.js with Microsoft Entra ID
- Secure httpOnly session cookies (no client-side token storage)
- Middleware-based route protection in `src/middleware.ts`
- Role-based access control (ADMIN, MANAGER, EMPLOYEE)

### Data Models (Prisma)
- **User**: Employees with roles, departments, and manager relationships
- **Department**: Organizational units
- **TimeOffRequest**: Requests with approval workflow states
- **TimeOffBalance**: Available days by type and year
- **RequestComment**: Comments on requests

### Service Layer Architecture
All business logic is contained in services (`src/lib/services/`):
- `user.service.ts` - User management and authentication
- `time-off-request.service.ts` - Request creation and approval workflow
- `time-off-balance.service.ts` - Balance tracking and updates
- `department.service.ts` - Department management

### API Structure
Next.js API routes in `/app/api/`:
- `/api/auth/[...nextauth]` - Authentication endpoints
- `/api/users` - User management
- `/api/departments` - Department operations
- Additional endpoints follow RESTful patterns

## Development Guidelines

### Environment Configuration
Required environment variables:
```
# Database
DATABASE_URL="postgresql://..."

# NextAuth
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="..."

# Microsoft Entra ID
AZURE_AD_CLIENT_ID="..."
AZURE_AD_CLIENT_SECRET="..."
AZURE_AD_TENANT_ID="..."
```

### Prisma Workflow
1. Schema is in `prisma/schema.prisma`
2. Generated client outputs to `src/lib/generated/prisma/`
3. Always run `npm run build` after schema changes (includes prisma generate)
4. Use migrations for schema changes: `npm run migrate:dev`

### Testing Strategy
- Write unit tests for all services using Vitest
- Use React Testing Library for component tests
- E2E tests with Playwright for critical user flows
- Run tests before marking any task as complete

### Code Style
- Follow existing patterns in the codebase
- Use services for business logic, not in components or API routes
- Leverage TypeScript's strict mode for type safety
- Component structure follows shadcn/ui patterns

### Git Workflow
- Feature branches off `main`
- Descriptive commit messages
- Run `npm run lint` before committing
- Ensure all tests pass before creating PRs

## Important Notes
- Prisma client has custom output path: `src/lib/generated/prisma/`
- Project uses ES modules (`"type": "module"` in package.json)
- Authentication tokens are never exposed to client-side code
- All date/time operations should use date-fns for consistency
- The project follows the Phase 1 roadmap focusing on MVP features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/workmani)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/workmani)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
