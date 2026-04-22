## concetrateaiquiz

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Canvas-style School Portal Platform** for the Concentrate.ai hiring assessment. The system models an educational SaaS platform with three user roles (Admin, Teacher, Student) supporting class management, lesson publication, assignment submission, and grading.

**Current Status (Session 13):**
- ✅ Backend API Complete - 42 endpoints, 95%+ test coverage
- ✅ Database layer with Kysely ORM
- ✅ JWT authentication with refresh token rotation
- ✅ 59 integration tests passing
- 🔄 Next: Google OAuth implementation

**Key Constraints:**
- Use **only** dependencies listed in `package.json` (may add Radix/shadcn UI components)
- **90%+ test coverage required** (100% aspirational, documented exceptions acceptable)
- **No `any` types allowed** - enforced by ESLint

**Documentation:**
- Full specifications: [docs/planning/SPECS.md](docs/planning/SPECS.md)
- Testing guide: [TESTING.md](TESTING.md)
- Session history: [docs/sessions/](docs/sessions/)
- Latest status: [docs/sessions/SESSION_13_STATUS.md](docs/sessions/SESSION_13_STATUS.md)

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 15, React 19, TailwindCSS, Radix UI |
| Backend | Node.js, Fastify, TypeScript, Zod |
| Database | PostgreSQL 17 with Kysely ORM |
| Caching | Redis |
| Testing | Vitest, @testing-library/react, Supertest, Playwright |

## Development Commands

### Initial Setup
```bash
npm install
docker-compose up -d  # Start PostgreSQL and Redis
```

### Database Services
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Check service health
docker-compose ps
```

**Database Connection:**
- PostgreSQL: `localhost:5432`
- Database: `concentrate-quiz`
- User/Password: `postgres/postgres`
- Redis: `localhost:6379`

### Testing
```bash
# Run all tests
npm run test

# View coverage report
npm run coverage

# Run specific test file
npm run test <path-to-test>

# Run E2E tests with Playwright
npm run test:e2e
```

**Testing Requirements:**
- 100% coverage enforced in CI/CD
- Unit tests for all service methods
- Integration tests for all API endpoints
- Component tests for key UI features
- E2E tests with Playwright for critical flows

### Development
```bash
npm run dev
```

### Code Quality
```bash
# Lint check (enforces no `any` types)
npm run lint

# Format code (Prettier with Tailwind plugin)
npm run format
```

## Architecture

### User Roles & Permissions

**Admin:**
- CRUD teacher groups
- CRUD users
- Suspend/unsuspend students and teachers

**Teacher:**
- CRUD classes
- Add/remove students from classes
- Publish assignments
- Grade submissions and provide feedback

**Student:**
- View enrolled classes and assignments
- Submit assignments
- View grades and feedback

### School Statistics API (`/api/v0/stats/`)

Public API endpoints for school-wide metrics:

| Endpoint | Method | Returns |
|----------|--------|---------|
| `/average-grades` | GET | Average grade across all classes |
| `/average-grades/:id` | GET | Average grade for specific class |
| `/teacher-names` | GET | List of all teacher names |
| `/student-names` | GET | List of all student names |
| `/classes` | GET | List of all classes |
| `/classes/:id` | GET | List of students in a class |

### Authentication

- **JWT tokens** with secure HTTP-only cookies
- All services must be protected
- Must integrate **at least 1 OAuth provider** (Google, Microsoft, or GitHub)

## Project Structure (To Be Implemented)

The project currently has only the starter configuration. The expected structure should follow a monorepo pattern or clear service separation for:

- Frontend (Next.js application)
- Backend API (Fastify server)
- Database migrations and schema (Kysely)
- Shared types and utilities
- Test suites (unit, integration, E2E)

## Code Standards

### TypeScript
- **No `any` types** - this is enforced by ESLint and will fail builds
- Use Zod for runtime validation
- Kysely for type-safe database queries

### Formatting
- Prettier with Tailwind CSS plugin
- Single quotes, 2-space tabs, 80 char width
- Trailing commas (ES5)

### React
- React 19 with hooks
- No prop-types (TypeScript only)
- `react-in-jsx-scope` disabled (Next.js handles this)

## Deployment Requirements

### Containerization
- All services must be containerized
- Single root-level Dockerfile to spin up entire stack
- Docker Compose for local development

### Production Deployment
- Deploy via Docker Compose on cloud instance
- Nginx reverse proxy
- SSL certificates via Certbot
- CI/CD pipeline must:
  - Run all tests
  - Build all services
  - Push to Docker Hub

## Extra Credit
Implement a chatbot that:
- Makes API calls to an LLM provider
- Has app-level context awareness
- Answers basic questions about the platform

## Submission Requirements
5-10 minute video covering:
- Application walkthrough (UI and features)
- Architecture decisions
- Testing approach
- Deployment setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BrianDai22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
