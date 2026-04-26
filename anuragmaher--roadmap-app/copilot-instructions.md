## roadmap-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack roadmap publishing application with public quarterly views and authenticated editing. It uses a React TypeScript frontend with a Node.js/Express backend and MongoDB for data persistence, plus Redis for caching.

## Development Commands

### Root Level Commands (from repository root)
```bash
npm run install-all          # Install dependencies for all packages
npm run dev                  # Start both backend and frontend in development mode
npm run build               # Build frontend and copy to build directory
npm start                   # Start frontend only
npm test                    # Run frontend tests
```

### Frontend Commands (from frontend/ directory)
```bash
npm run lint                # Run ESLint on TypeScript files
npm run lint:strict         # Run ESLint with max-warnings 0
npm run lint:fix            # Auto-fix ESLint issues
npm run typecheck           # TypeScript type checking only
npm run code-quality        # Run both lint and typecheck
npm run test                # Run Jest tests
npm run pre-commit          # Full quality check and tests
```

### Backend Commands (from backend/ directory)
```bash
npm run dev                 # Start backend with nodemon (auto-reload)
npm start                   # Start backend in production mode
```

## Architecture

### Full-Stack Structure
- **Frontend**: React 19 with TypeScript, React Router v7, Create React App
- **Backend**: Node.js with Express, MongoDB via Mongoose, Redis for caching
- **Authentication**: JWT-based with bcrypt password hashing
- **API**: RESTful endpoints under `/api/` routes
- **Deployment**: Vercel-ready with serverless API routes

### Frontend Architecture (src/)
```
components/          # Reusable UI components (<30 lines each per guidelines)
pages/              # Route components (<200 lines each per guidelines)  
hooks/              # Custom React hooks for stateful logic
contexts/           # React contexts for global state
services/           # API calls and external service integrations
utils/              # Pure utility functions
types/              # TypeScript type definitions
styles/             # CSS and styling files
```

### Backend Architecture (src/)
```
controllers/        # Request handlers and business logic
routes/            # Express route definitions
models/            # Mongoose schemas and models
middleware/        # Express middleware (auth, validation, etc.)
services/          # Business logic and external service integrations
server.js          # Main server entry point
```

### Key Technologies
- **Frontend**: React 19, TypeScript 4.9, React Router 7.8, Axios
- **Backend**: Express 4.18, Mongoose 7.5, Redis 4.6, JWT, bcryptjs
- **Development**: ESLint, Husky, lint-staged, Prettier, nodemon
- **Testing**: Jest, React Testing Library
- **Deployment**: Vercel with serverless functions

## Code Quality Guidelines

### File Size Limits (enforced via ESLint)
- **Components**: Max 200 lines (warned at 250)
- **Functions**: Max 30 lines (warned at 80)
- **Complexity**: Max 8 cyclomatic complexity
- **Parameters**: Max 3 parameters per function

### Code Organization Principles
1. **Single Responsibility**: Each file/component should have ONE clear purpose
2. **DRY Principle**: Extract common logic into hooks/utilities
3. **Component Decomposition**: Split large components into smaller, focused pieces
4. **Custom Hooks**: Extract stateful logic when patterns repeat

### ESLint Rules (frontend/.eslintrc.js)
- No console logs except warn/error
- Prefer const over let/var
- Enforce semicolons and single quotes
- Self-closing components
- No useless fragments
- Max lines per file/function warnings

## Environment Setup

### Backend Environment (.env in backend/)
```
MONGODB_URI=mongodb://localhost:27017/roadmap-app
JWT_SECRET=your_jwt_secret_here
PORT=5000
REDIS_URL=redis://localhost:6379
DEFAULT_TENANT=your-default-tenant  # For localhost testing
```

### Redis Setup
- **Local Development**: Install via `brew install redis` (macOS) or use Docker
- **Production**: Use Redis Cloud, AWS ElastiCache, or Upstash
- **Cache TTL**: 5 minutes for home page data
- **Cache Keys**: Pattern `home_data:{tenantId}:{hostname}`
- **Graceful Degradation**: App works without Redis (direct DB queries)

## Data Models

### Core Entities
- **Roadmaps**: Main roadmap documents with tenants and metadata
- **Items**: Individual roadmap items with quarters, tags, and content
- **Users**: Authentication and user management
- **Tenants**: Multi-tenancy support for different roadmap instances

### Quarters System
- Public views organized by quarters (Q1, Q2, Q3, Q4)
- Items assigned to specific quarters
- Quarterly filtering and display logic

## Testing

### Frontend Testing
```bash
cd frontend && npm test           # Run all tests
cd frontend && npm test -- --watchAll=false  # Run once without watch
```

### Testing Libraries Available
- Jest for unit testing
- React Testing Library for component testing
- @testing-library/user-event for user interaction testing

## Authentication Flow

1. **Login**: POST `/api/auth/login` with email/password
2. **JWT Token**: Returned and stored for authenticated requests
3. **Protected Routes**: Middleware validates JWT tokens
4. **Public Access**: Roadmap viewing is public, editing requires auth

## Performance Optimizations

### Caching Strategy
- **Server-side**: Redis caching for home page data (5min TTL)
- **Auto-invalidation**: Cache cleared on data changes
- **Benefits**: ~90% faster response times on cached requests

### Build Optimization
- React build optimization via Create React App
- Static asset optimization and caching
- Production-ready builds copied to `/build` directory

## Development Workflow

### Code Quality Enforcement
1. **Pre-commit Hooks**: Husky + lint-staged for automatic quality checks
2. **ESLint**: Enforces code style and complexity rules
3. **TypeScript**: Strict type checking with `npm run typecheck`
4. **File Size Monitoring**: Use `npm run file-sizes` to track component sizes

### Common Development Tasks
1. **Adding Components**: Follow size limits, single responsibility
2. **API Integration**: Use services/ directory for API calls
3. **State Management**: Extract complex state to custom hooks
4. **Styling**: Component-scoped CSS with utility classes
5. **Type Safety**: Define types in types/ directory

## Deployment

### Vercel Configuration
- **API Routes**: Backend deployed as serverless functions under `/api/`
- **Rewrites**: All `/api/*` requests routed to serverless backend
- **Environment Variables**: Set MONGODB_URI, JWT_SECRET, REDIS_URL in Vercel
- **Build**: Frontend built and served statically

### Production Checklist
1. Configure MongoDB connection string
2. Set up Redis hosting (Redis Cloud, etc.)
3. Generate secure JWT secret
4. Configure Vercel environment variables
5. Test API routes and database connectivity

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anuragmaher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
