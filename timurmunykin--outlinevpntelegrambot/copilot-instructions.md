## outlinevpntelegrambot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based VPN management application that integrates with Outline VPN servers. The application provides a comprehensive web interface for managing VPN access keys, users, billing system, and server administration.

## Architecture

The project is a full-stack web application with the following structure:

### Backend (`/backend`)
- **Express.js** REST API with TypeScript
- **Prisma ORM** with PostgreSQL database
- **JWT Authentication** with role-based access control
- **OAuth 2.0** provider for third-party integrations
- **Billing System** with subscription management and automated billing
- **Cron Jobs** for scheduled tasks (daily billing)

### Frontend (`/frontend`)
- **React** with TypeScript and Vite
- **Material-UI (MUI)** for consistent design system
- **React Router** for client-side routing
- **Zustand** for state management

### Key Components

#### Backend Services
- **VPN Service**: Manages Outline VPN server integration
- **Billing Service**: Handles subscriptions, balance management, traffic limiting
- **Cron Service**: Automated daily billing and maintenance tasks
- **User Management**: Authentication, authorization, user limits

#### Frontend Pages
- **Dashboard**: Overview and quick actions
- **VPN Keys**: Create, manage, and monitor VPN access keys
- **Billing**: Subscription management (user view) and admin billing tools
- **Users**: User management (admin only)
- **Settings**: Global application configuration (admin only)

## Environment Variables Required

The application requires these environment variables in a `.env` file:
- `DATABASE_URL`: PostgreSQL connection string
- `JWT_SECRET`: Secret key for JWT token signing
- `OUTLINE_API_URL`: Outline VPN server API URL
- `OUTLINE_API_FINGERPRINT`: Outline VPN server fingerprint for authentication

## Development Commands

### Development Workflow
For quick UI development and testing with hot reload:
```bash
# Start development environment (recommended for UI changes)
./dev.sh dev
# This starts:
# - Backend and database in Docker containers
# - Frontend with Vite dev server (hot reload) in Docker
# - Frontend available at: http://localhost:3000
# - Backend API at: http://localhost:3001/api
# - Database at: localhost:5433

# Login credentials:
# Admin: admin@test.local / admin123
# User: user@test.local / user123
```

### Production Testing
For full production builds:
```bash
# Full production build and test
./dev.sh test
# - Builds all services with production Dockerfiles
# - Available at: http://localhost:8080
```

### Manual Development
```bash
# Backend development (manual)
cd backend
npm install
npm run dev

# Frontend development (manual)
cd frontend
npm install
npm run dev

# Database operations
npx prisma migrate dev
npx prisma db seed
npx prisma studio
```

### CRITICAL: Agent-Based Workflow (HIGHEST PRIORITY)
**ALL USER REQUESTS MUST GO THROUGH SPECIALIZED AGENTS - NEVER WORK DIRECTLY**

1. **dev-team-coordinator** - ALWAYS USE FIRST for any user request
   - Acts as team lead/manager for all development tasks
   - Analyzes requirements and delegates to appropriate specialists
   - Coordinates complex multi-step features and bug fixes
   - Tracks project progress across all agents

2. **Specialized Agents** (delegated by coordinator):
   - **frontend-feature-developer**: New UI features, components, pages
   - **frontend-bug-fixer**: UI bugs, styling issues, component fixes
   - **typescript-backend-developer**: New API endpoints, backend features
   - **backend-bug-hunter**: Backend bugs, API issues, server problems
   - **frontend-test-writer**: Frontend unit/integration tests
   - **backend-test-writer**: Backend API/service tests
   - **cicd-docker-engineer**: Docker, CI/CD, deployment configuration

3. **Workflow**:
   ```
   User Request → dev-team-coordinator → Specialist Agent(s) → Complete Task
   ```

**NEVER bypass this workflow - always start with dev-team-coordinator agent!**

### Important Notes for Claude Code
- **ALWAYS use `./dev.sh dev` for development workflow** - this enables hot reload for frontend changes
- **NEVER use full production builds for UI testing** - use dev environment for quick iterations
- After making frontend changes, they will be automatically reflected at http://localhost:3000
- Only use `./dev.sh test` when you need to test the full production build
- **Git operations (add, commit, push) are handled by the user** - Claude should not perform git commands

## Production Deployment

### Local to Registry Deployment
```bash
# Build and push images to Docker registry
./dev.sh deploy
# This builds production images and pushes them to the configured registry
# Uses docker-compose.registry.yml for registry configuration
```

### Server Deployment (after registry push)
After pushing images to registry, on the production server:
```bash
# Pull latest code and Docker images
git pull origin master
docker compose -f docker-compose.server.yml pull
docker compose -f docker-compose.server.yml up -d --force-recreate

# Or use deployment script if available
./deploy.sh
```

### Development Script Commands
```bash
./dev.sh build    # Build images locally
./dev.sh up       # Start local development environment
./dev.sh logs     # Show logs (optionally specify service)
./dev.sh stop     # Stop local environment
./dev.sh clean    # Clean up all local data
./dev.sh seed     # Seed database with test users
./dev.sh test     # Test locally before deploying
./dev.sh deploy   # Build and push to production registry
```

### Manual Production Deployment
```bash
# Copy and configure environment
cp .env.production .env
# Edit .env with your settings

# Deploy with Docker Compose
./deploy.sh

# Or manual deployment
docker-compose -f docker-compose.prod.yml up -d
```

## Key Dependencies

### Backend
- **express**: Web application framework
- **prisma**: Database ORM and query builder
- **jsonwebtoken**: JWT authentication
- **node-cron**: Scheduled task execution
- **outlinevpn-api**: Outline VPN management API client

### Frontend
- **react**: UI library
- **@mui/material**: Material Design components
- **react-router-dom**: Client-side routing
- **zustand**: State management
- **axios**: HTTP client

## Code Structure Notes

- **Type Safety**: Full TypeScript coverage across backend and frontend
- **Authentication**: JWT-based with role-based access control (USER/ADMIN)
- **Database**: Prisma ORM with PostgreSQL, migrations and seeding
- **API Design**: RESTful endpoints with Swagger documentation
- **Security**: Rate limiting, CORS, helmet security headers
- **Billing**: Automated subscription management with traffic limiting
- **Error Handling**: Comprehensive error handling with user-friendly messages
- **Production Ready**: Docker containers, nginx proxy, automatic backups

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TimurMunykin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
