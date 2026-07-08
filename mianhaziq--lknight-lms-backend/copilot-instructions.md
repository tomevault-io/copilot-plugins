## lknight-lms-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LKnight is a Learning Management System (LMS) backend API built with Express.js 5, Prisma 7, and PostgreSQL. It provides REST APIs for course management, user authentication, enrollments, and admin analytics.

**Requirements**: Node.js 22.12+ (Prisma 7 requirement)

## Commands

```bash
# Development
npm run dev          # Start with nodemon (hot reload)
npm start            # Start production server

# Database
npx prisma generate        # Generate Prisma client after schema changes
npx prisma migrate dev     # Create and run migrations (dev)
npx prisma migrate deploy  # Run migrations (production)
npx prisma studio          # Open Prisma Studio GUI
```

## Architecture

### Directory Structure
- `server.js` - Express app entry point, route mounting, middleware setup
- `src/config/db.js` - Prisma client singleton with PrismaPg adapter
- `src/config/swagger.js` - OpenAPI/Swagger configuration
- `src/controllers/` - Request handlers for each domain
- `src/routes/` - Express routers with Swagger JSDoc annotations
- `src/middleware/` - Auth verification and error handling
- `prisma/schema.prisma` - Database schema and enums
- `prisma.config.ts` - Prisma 7 configuration (datasource URL, migrations path)
- `docs/API_DOCUMENTATION.md` - Complete API reference

### Prisma 7 Configuration

This project uses Prisma 7 with the driver adapter pattern. Key differences from Prisma 6:

1. **No `url` in schema.prisma** - The datasource URL is defined in `prisma.config.ts`, not in the schema
2. **Adapter pattern** - `src/config/db.js` uses `@prisma/adapter-pg` to create the connection
3. **Config file** - `prisma.config.ts` is required for CLI operations (migrations, generate)

### Key Patterns

**Database Access**: Import Prisma client from `src/config/db.js`:
```javascript
const prisma = require('../config/db');
```

**Authentication Middleware** (in `src/middleware/auth.js`):
- `verifyToken` - Validates JWT and sets `req.userId`, `req.userRole`
- `verifyAdmin` - Checks if token belongs to an Admin record
- `verifyInstructorOrAdmin` - Allows access for instructors or admins

**Response Format**: All endpoints return consistent JSON:
```javascript
{ success: true, data: {...}, message: "..." }
{ success: false, message: "Error description" }
```

**Error Handling**: Controllers use `next(error)` to delegate to `src/middleware/errorHandler.js`, which handles Prisma error codes (P2002 for duplicates, P2025 for not found).

### Nested Routes

Modules and lessons use nested routing under their parent resources:
- `/api/courses/:courseId/modules` - Module CRUD under courses
- `/api/modules/:moduleId/lessons` - Lesson CRUD under modules
- `/api/modules/:id` and `/api/lessons/:id` - Standalone routes for direct access

Routes export both the nested router (default) and `standaloneRouter` for direct ID access.

### Data Models

Core entities: User, Admin, Category, Course, Module, Lesson, Enrollment, Settings

Enums defined in Prisma schema: `UserRole`, `UserStatus`, `CourseLevel`, `CourseStatus`, `EnrollmentStatus`

## Environment Variables

Required in `.env`:
- `DATABASE_URL` - PostgreSQL connection string
- `JWT_SECRET` - Secret for signing JWT tokens
- `JWT_EXPIRES_IN` - Token expiration (default: 7d)
- `GOOGLE_CLIENT_ID` - For Google OAuth
- `SMTP_*` variables - For email sending (password reset)
- `FRONTEND_URL` - For password reset links

## API Documentation

- Swagger UI available at `/api-docs` when server is running
- Full API reference in `docs/API_DOCUMENTATION.md`

## Deployment (Railway)

Configured for Railway deployment using Docker:

- `Dockerfile` - Production build with node:22-slim, installs OpenSSL for Prisma
- `railway.toml` - Railway config (uses Dockerfile builder, healthcheck on `/`)
- `start.sh` - Startup script: runs migrations then starts server

Environment variables to set in Railway:
- `DATABASE_URL` - Use Railway's PostgreSQL internal URL
- `JWT_SECRET`, `JWT_EXPIRES_IN`
- `ALLOWED_ORIGINS` - Comma-separated frontend URLs for CORS
- `GOOGLE_CLIENT_ID`, `SMTP_*` variables as needed
- Optional security: `ENABLE_API_DOCS=true` to expose `/api-docs` in production; `ENABLE_SWAGGER_INTERACTIVE=true` for Swagger "Try it out"; `TRUST_PROXY=1` if not using default production trust proxy; `RATE_LIMIT_AUTH_MAX`, `RATE_LIMIT_API_MAX` to tune limits

---
> Source: [MianHaziq/LKnight-Lms-backend](https://github.com/MianHaziq/LKnight-Lms-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
