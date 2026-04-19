## budget-reduction-tracking

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Budget Reduction Tracking is a full-stack web application for tracking debt reduction and credit management progress. The system provides account management, transaction tracking, predictive analytics, and data visualization to help users achieve financial freedom.

**Tech Stack**: React 19 + TypeScript (frontend), Node.js/Express + Prisma + PostgreSQL (backend)

## Common Commands

### Backend Development
```bash
cd backend

# Development
npm run dev                  # Start dev server with hot reload (port 3001)
npm test                     # Run tests
npm run test:watch           # Run tests in watch mode
npm run test:coverage        # Generate coverage report

# Database (Prisma)
npx prisma migrate dev       # Create and apply migration
npx prisma migrate deploy    # Apply migrations (production)
npx prisma generate          # Generate Prisma Client
npx prisma studio            # Open database GUI
npx prisma migrate reset     # Reset database (dev only - destructive)

# Build
npm run build                # Build TypeScript
npm start                    # Start production server
```

### Frontend Development
```bash
cd frontend

# Development
npm run dev                  # Start Vite dev server (port 5173)
npm test                     # Run tests with Vitest
npm run test:watch           # Run tests in watch mode
npm run test:coverage        # Generate coverage report

# Build
npm run build                # Build for production
npm run preview              # Preview production build
npm run type-check           # TypeScript type checking
npm run lint                 # Run ESLint
```

### Running a Single Test
```bash
# Backend - Jest
cd backend
npm test -- auth.service.test.ts          # Specific test file
npm test -- --testNamePattern="login"     # Tests matching pattern

# Frontend - Vitest
cd frontend
npm test -- AccountList.test.tsx          # Specific test file
npm test -- --grep "should render"        # Tests matching pattern
```

## Architecture & Code Structure

### Three-Layer Architecture

**Backend** (`/backend/src/`):
```
controllers/     → Request handlers (thin, delegate to services)
services/        → Business logic (core functionality)
validators/      → Zod schemas for request validation
middleware/      → Auth, error handling, logging
routes/          → Express route definitions
```

**Frontend** (`/frontend/src/`):
```
pages/           → Top-level page components
components/      → Reusable UI components
hooks/           → Custom React hooks
services/        → API client layer (axios)
types/           → TypeScript type definitions
```

### Data Model (Prisma Schema)

**Core Entities**:
- `User` - Authentication and user management
- `Account` - Credit cards, loans, mortgages with interest tracking
- `Transaction` - Payments, charges, adjustments, interest
- `Snapshot` - Historical balance tracking for trend analysis

**Key Relationships**:
- User → Accounts (one-to-many, cascade delete)
- Account → Transactions (one-to-many, cascade delete)
- Account → Snapshots (one-to-many, cascade delete)

### Authentication Flow

JWT-based authentication with refresh tokens:
1. Login: POST `/api/auth/login` → returns access token (15min) + refresh token (7d)
2. Protected endpoints require Bearer token in Authorization header
3. Token refresh: POST `/api/auth/refresh` with refresh token
4. Auth middleware (`backend/src/middleware/auth.middleware.ts`) validates JWT
5. Frontend stores tokens via axios interceptors and React Query

### API Patterns

**Standard Response Format**:
```typescript
// Success
{ success: true, data: {...} }

// Error
{ success: false, error: { message: "...", code: "..." } }
```

**Endpoint Naming**:
- Auth: `/api/auth/login`, `/api/auth/register`
- Accounts: `/api/accounts`, `/api/accounts/:id`
- Transactions: `/api/accounts/:accountId/transactions`
- Analytics: `/api/analytics/overview`, `/api/analytics/projections`
- Health: `/api/health` (unauthenticated)

### Interest Calculations

**Daily Interest Accumulation**:
```typescript
const dailyRate = annualRate / 365;
const dailyInterest = balance * dailyRate;
```

**Payoff Projections**: Located in `backend/src/services/projection.service.ts`. Algorithm simulates monthly payments with compound interest to estimate payoff timeline and total interest cost.

### Chart Visualization

**Inverted Progress Display**: Charts show debt reduction as upward progress (decreasing balance = positive trend). Implementation uses Chart.js with inverted Y-axis or calculated reduction amounts.

Primary charts:
- Balance reduction timeline (line chart)
- Interest forecast (area chart)
- Payment distribution (pie chart)

## Database Conventions

**All IDs are UUIDs** (not integers)

**Decimal Fields**: Use Prisma's `Decimal` type for financial data (precision: 12,2)
```typescript
// Correct
currentBalance: Decimal @db.Decimal(12, 2)

// Incorrect - never use Float for money
currentBalance: Float
```

**Timestamps**: All models have `createdAt` and `updatedAt` (except Snapshot which only has `createdAt`)

**Indexes**: Critical indexes on:
- `User.email`
- `Account.userId`, `Account.accountType`
- `Transaction.accountId`, `Transaction.transactionDate`
- `Snapshot.accountId`, `Snapshot.snapshotDate`

## Environment Setup

**Prerequisites**:
- Node.js 20 LTS
- PostgreSQL 16
- npm 10+

**First-Time Setup**:
```bash
# Backend
cd backend
npm install
cp .env.example .env
# Edit .env with database credentials
npx prisma migrate dev
npx prisma generate

# Frontend
cd frontend
npm install
cp .env.example .env
# Edit VITE_API_URL if needed (default: http://localhost:3001)
```

**Required Environment Variables**:

Backend `.env`:
- `DATABASE_URL` - PostgreSQL connection string
- `JWT_SECRET` - Secret for access tokens
- `REFRESH_TOKEN_SECRET` - Secret for refresh tokens
- `PORT` - Default 3001
- `CORS_ORIGIN` - Frontend URL (default: http://localhost:5173)

Frontend `.env`:
- `VITE_API_URL` - Backend API URL (default: http://localhost:3001)

## Testing Strategy

**Backend**: Jest with Supertest for API integration tests. Test database uses separate `.env.test` configuration.

**Frontend**: Vitest + React Testing Library for component tests.

**Coverage Goals**:
- Backend: 80%+ coverage
- Frontend: 70%+ coverage
- Critical paths (auth, financial calculations): 100%

When writing tests:
- Use descriptive test names: `"should calculate correct daily interest for APR"`
- Mock external dependencies (database, API calls)
- Test edge cases (zero balance, negative amounts, invalid dates)
- Test error handling paths

## Code Quality Standards

### TypeScript
- Strict mode enabled
- All functions must have explicit return types
- No `any` types (use `unknown` if truly dynamic)
- Use Zod for runtime validation at API boundaries

### Backend Patterns
- Controllers are thin - delegate to services
- Services contain business logic
- Use Prisma transactions for multi-step operations
- Error handling via custom error classes and middleware

### Frontend Patterns
- Functional components with hooks only
- React Query for server state management
- React Hook Form + Zod for form validation
- Tailwind CSS for styling
- Component files co-located with tests

### Security
- All user input validated (Zod schemas)
- SQL injection prevented via Prisma ORM
- XSS protection via React's built-in sanitization
- Passwords hashed with bcrypt (12 rounds)
- Rate limiting enabled on auth endpoints

## Deployment

**Production Target**: Proxmox LXC container with native services (not Docker)

**Stack**:
- Nginx (serves frontend + proxies API)
- PM2 (Node.js process management)
- PostgreSQL (local database)
- Nginx Proxy Manager (separate LXC, handles SSL/reverse proxy)

**External Access Flow**:
```
Internet → Cloudflare → UniFi Gateway → Nginx Proxy Manager → App Container
```

See `docs/DEPLOYMENT.md` for complete setup instructions.

## Key Files to Review

When starting work on this project, prioritize reading:
1. `ARCHITECTURE.md` - Complete system design and technical decisions
2. `backend/prisma/schema.prisma` - Data model and relationships
3. `backend/src/services/` - Core business logic
4. `frontend/src/services/api.ts` - API client setup
5. `docs/API.md` - Complete API reference

## Common Pitfalls

1. **Don't use `Float` for currency** - Always use `Decimal` with Prisma
2. **Don't forget to run `npx prisma generate`** after schema changes
3. **Don't skip validation** - Validate on both frontend and backend
4. **Don't hardcode URLs** - Use environment variables
5. **Don't commit `.env` files** - Use `.env.example` templates
6. **Account for timezone differences** - Store dates in UTC, display in local time
7. **Test with realistic data** - Financial calculations need precision testing

## Git Workflow

Branch naming: `feature/description` or `fix/description`

Commit message format:
```
type(scope): brief description

Longer explanation if needed

- Bullet points for details
```

Types: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`

## Additional Resources

- `README.md` - Project overview and quick start
- `ARCHITECTURE.md` - Comprehensive system architecture
- `QUICK_START.md` - Development setup guide
- `docs/DATABASE.md` - Database schema details
- `docs/DEVELOPMENT.md` - Development workflow
- `docs/COMPONENTS.md` - Frontend component documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmcguire80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
