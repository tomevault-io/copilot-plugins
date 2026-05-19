## take-home

> This document provides comprehensive guidance for AI assistants working with this codebase.

# CLAUDE.md - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with this codebase.

## Project Overview

This is a **fullstack monorepo** built with **Turborepo**, featuring a **Next.js 15** frontend and **NestJS** backend with **Prisma ORM**. The architecture follows a **feature-based** pattern, emphasizing maintainability, scalability, and productivity. The entire stack runs in Docker for consistent development environments.

**Key Characteristics:**
- Monorepo with pnpm workspaces
- Feature-based architecture (domain-driven organization)
- TypeScript throughout
- Comprehensive test coverage
- Redis caching layer
- JWT-based authentication with HTTP-only cookies
- SSR with React Query hydration

## Repository Structure

```
/home/user/take-home/
├── apps/
│   ├── backend/          # NestJS API (port 3001)
│   ├── web/              # Next.js frontend (port 3000)
│   └── docs/             # Documentation site
├── packages/
│   ├── ui/               # Shared React components
│   ├── eslint-config/    # ESLint configurations
│   └── typescript-config/# TypeScript configurations
├── docker-compose.yml    # Full stack orchestration
├── turbo.json            # Turborepo configuration
└── .env                  # Docker environment variables
```

### Backend Structure (`apps/backend/`)

```
apps/backend/
├── prisma/
│   ├── schema.prisma     # Database schema (User model)
│   └── seed.ts           # Database seeding script
├── src/
│   ├── auth/             # Authentication module
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── auth.guard.ts
│   ├── users/            # Users feature (feature-based)
│   │   ├── controllers/users.controller.ts
│   │   ├── services/users.service.ts
│   │   └── dto/          # Data transfer objects
│   ├── prisma/           # Prisma module (global)
│   │   ├── prisma.module.ts
│   │   └── prisma.service.ts
│   ├── redis/            # Redis caching module
│   │   ├── redis.module.ts
│   │   └── redis.service.ts
│   ├── app.module.ts
│   └── main.ts           # Entry point (port 3000 internal)
├── test/                 # E2E tests
│   ├── *.e2e-spec.ts
│   └── jest-e2e.json
├── Dockerfile
├── init.sh               # Docker initialization script
└── .env.local.example    # Local development env template
```

### Frontend Structure (`apps/web/`)

```
apps/web/
├── app/
│   ├── auth/             # Authentication pages (feature-based)
│   │   ├── login/
│   │   │   ├── page.tsx
│   │   │   └── page.test.tsx
│   │   └── register/
│   │       ├── page.tsx
│   │       └── page.test.tsx
│   ├── users/            # Users feature (feature-based)
│   │   ├── components/
│   │   │   ├── UserList.tsx
│   │   │   ├── UserList.test.tsx
│   │   │   ├── UserForm.tsx
│   │   │   ├── UserForm.test.tsx
│   │   │   ├── UserDeleteButton.tsx
│   │   │   └── UserDeleteButton.test.tsx
│   │   ├── [id]/page.tsx    # Edit user (dynamic route)
│   │   ├── new/page.tsx     # Create user
│   │   └── page.tsx         # Users list (SSR + hydration)
│   ├── api/              # API routes (proxies to backend)
│   │   └── auth/
│   │       ├── login/route.ts
│   │       ├── logout/route.ts
│   │       └── register/route.ts
│   ├── components/       # Shared components
│   │   ├── Header.tsx
│   │   └── LogoutButton.tsx
│   ├── layout.tsx        # Root layout (QueryClientProvider)
│   ├── page.tsx          # Home (redirects to /auth/register)
│   ├── providers.tsx     # React Query setup
│   └── globals.css
├── lib/
│   ├── services/
│   │   ├── userService.ts      # API client functions
│   │   └── userService.test.ts
│   └── queryClient.ts    # React Query client factory
├── middleware.ts         # Route protection
├── Dockerfile
├── jest.config.ts
└── .env.local.example    # Local development env template
```

## Tech Stack

### Backend
- **Framework:** NestJS 11
- **ORM:** Prisma 6.6.0
- **Database:** PostgreSQL 16 (Docker)
- **Cache:** Redis 7.2 (ioredis)
- **Auth:** JWT (jsonwebtoken) + bcrypt
- **Testing:** Jest + Supertest
- **Language:** TypeScript 5.8.2
- **Build:** NestJS CLI with SWC

### Frontend
- **Framework:** Next.js 15.3.0 (App Router)
- **React:** 19.1.0
- **Data Fetching:** TanStack React Query v5.74.4
- **Styling:** Tailwind CSS 3.4.1
- **Testing:** Jest + React Testing Library
- **Build:** Turbopack (dev)
- **Language:** TypeScript 5.8.2

### Infrastructure
- **Monorepo:** Turborepo 2.5.0
- **Package Manager:** pnpm 9.0.0 (required)
- **Containers:** Docker + Docker Compose
- **Node:** 18+ (20-alpine in Docker)

## Development Workflows

### Essential Commands

**From repository root:**

```bash
# Install all dependencies
pnpm install

# Start entire stack (Docker)
docker-compose up -d

# Stop entire stack
docker-compose down

# Development mode (all apps)
pnpm dev

# Build all apps
pnpm build

# Run all linters
pnpm lint

# Format code
pnpm format

# Type check all apps
pnpm check-types
```

**Backend-specific (from root):**

```bash
# Run backend tests (unit + integration)
pnpm --filter backend test

# Run E2E tests (requires Docker running)
pnpm --filter backend test:e2e

# Test with coverage
pnpm --filter backend test:cov

# Generate Prisma client for Docker
pnpm --filter backend prisma:generate:docker

# Generate Prisma client for local VSCode
pnpm --filter backend prisma:generate:local

# Run migrations (Docker)
pnpm --filter backend prisma:migrate:docker

# Run migrations (local)
pnpm --filter backend prisma:migrate:local

# Seed database
pnpm --filter backend prisma:seed

# Start backend dev server
pnpm --filter backend start:dev
```

**Frontend-specific (from root):**

```bash
# Run frontend tests
pnpm --filter web test

# Run tests with coverage
pnpm --filter web test:coverage

# Start frontend dev server
pnpm --filter web dev

# Build frontend
pnpm --filter web build

# Lint frontend
pnpm --filter web lint

# Type check frontend
pnpm --filter web check-types
```

### Environment Setup

**CRITICAL:** The project requires **different environment variables** for Docker vs local development.

#### 1. Copy environment templates:

```bash
cp .env.example .env
cp apps/backend/.env.local.example apps/backend/.env.local
cp apps/web/.env.local.example apps/web/.env.local
```

#### 2. Environment Variables

**Root `.env` (for Docker):**
```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=app
DATABASE_URL=postgres://postgres:postgres@postgres:5432/app
REDIS_HOST=redis
REDIS_PORT=6379
JWT_SECRET=secret-example
```

**`apps/backend/.env.local` (for local development):**
```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=app
DATABASE_URL=postgres://postgres:postgres@localhost:5432/app
REDIS_HOST=localhost
REDIS_PORT=6379
JWT_SECRET=secret-example
```

**`apps/web/.env.local`:**
```env
NEXT_PUBLIC_API_URL=http://localhost:3001
```

**Key Differences:**
- Docker uses service names (`postgres`, `redis`, `backend`)
- Local uses `localhost`
- Backend internal port is `3000`, exposed as `3001`

### Docker Services

```yaml
services:
  postgres:  # Port 5432
  redis:     # Port 6379
  backend:   # Port 3001 (maps to internal 3000)
  web:       # Port 3000
```

**Backend initialization** (`apps/backend/init.sh`):
1. Install dependencies
2. Run Prisma migrations
3. Generate Prisma client
4. Seed database
5. Start dev server

### Prisma Workflow

**IMPORTANT:** Prisma client must be generated in two contexts:

1. **Inside Docker** (for runtime):
   ```bash
   pnpm --filter backend prisma:generate:docker
   ```

2. **Outside Docker** (for VSCode IntelliSense):
   ```bash
   pnpm --filter backend prisma:generate:local
   ```

**Custom Prisma Output Path:**
```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../node_modules/@generated/prisma"
}
```

This non-standard path enables both Docker and local development to work correctly.

## Code Conventions

### File Naming

**Backend (NestJS):**
- Controllers: `*.controller.ts` (e.g., `users.controller.ts`)
- Services: `*.service.ts` (e.g., `users.service.ts`)
- Modules: `*.module.ts` (e.g., `users.module.ts`)
- DTOs: `*.dto.ts` (e.g., `create-user.dto.ts`)
- Guards: `*.guard.ts` (e.g., `auth.guard.ts`)
- Unit tests: `*.spec.ts` (co-located with source)
- E2E tests: `*.e2e-spec.ts` (in `test/` directory)

**Frontend (Next.js App Router):**
- Pages: `page.tsx` (App Router convention)
- Layouts: `layout.tsx` (App Router convention)
- API routes: `route.ts` (App Router convention)
- Components: `PascalCase.tsx` (e.g., `UserList.tsx`)
- Tests: `*.test.tsx` or `*.test.ts` (co-located)
- Services: `camelCase.ts` (e.g., `userService.ts`)

### Directory Organization

**Feature-Based Structure** (both frontend and backend):

Each feature/domain has its own directory containing all related code:

```
users/
├── components/         # UI components (frontend)
├── services/           # Business logic (backend)
├── controllers/        # HTTP handlers (backend)
├── dto/                # Data transfer objects (backend)
└── *.test.tsx          # Tests (co-located)
```

**Benefits:**
- High cohesion within features
- Easy to locate related code
- Scales well as features grow
- Clear boundaries between domains

### Import Patterns

**Backend:**
```typescript
// Path alias for source files
import { PrismaService } from '@/prisma/prisma.service';

// Generated Prisma client
import { PrismaClient } from '@generated/prisma';

// Relative imports within features
import { CreateUserDto } from '../dto/create-user.dto';
```

**Frontend:**
```typescript
// Path alias (@ = root)
import { userService } from '@/lib/services/userService';
import Header from '@/app/components/Header';

// Relative imports for co-located files
import UserForm from './UserForm';
```

### TypeScript Configuration

Three shared configs in `packages/typescript-config/`:
- `base.json` - Common settings
- `nextjs.json` - Next.js specific
- `react-library.json` - React library packages

All configs enforce strict type checking.

### Code Style

**Prettier** (enforced):
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false
}
```

**ESLint** (shared in `packages/eslint-config/`):
- TypeScript ESLint recommended rules
- React hooks rules
- Next.js specific rules
- Prettier integration

## API Architecture

### Backend Endpoints

**Authentication** (no guard required):
```
POST /auth/register    # Create account + set JWT cookie
POST /auth/login       # Login + set JWT cookie
POST /auth/logout      # Clear JWT cookie
```

**Users** (requires AuthGuard):
```
GET    /users          # List all users (Redis cached)
GET    /users/:id      # Get single user (Redis cached)
POST   /users          # Create user
PUT    /users/:id      # Update user (invalidates cache)
DELETE /users/:id      # Delete user (invalidates cache)
```

### Authentication Flow

1. User logs in via `POST /auth/login`
2. Backend verifies credentials with bcrypt
3. JWT token generated and set as HTTP-only cookie
4. `AuthGuard` validates JWT on protected routes
5. Decoded user attached to `request.user`

**Key Files:**
- `apps/backend/src/auth/auth.guard.ts` - JWT validation
- `apps/backend/src/auth/auth.service.ts` - Login/register logic
- `apps/web/middleware.ts` - Frontend route protection

### Frontend API Integration

**Pattern:** Next.js API routes proxy authentication requests to forward cookies properly.

**API Proxies** (`apps/web/app/api/auth/*/route.ts`):
```typescript
// Forwards requests to backend, preserves cookies
export async function POST(request: Request) {
  const body = await request.json();
  const response = await fetch(`http://backend:3000/auth/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  });
  // Forward Set-Cookie headers
  return new Response(response.body, {
    status: response.status,
    headers: response.headers,
  });
}
```

**Service Layer** (`apps/web/lib/services/userService.ts`):
```typescript
// All requests include credentials for cookies
export const getUsers = async (): Promise<User[]> => {
  const response = await fetch(`${API_URL}/users`, {
    credentials: 'include',  // CRITICAL for cookies
  });
  return response.json();
};
```

**SSR Consideration:**
- Server-side: Use `http://backend:3000` (Docker service name)
- Client-side: Use `process.env.NEXT_PUBLIC_API_URL` (http://localhost:3001)
- Service layer handles this automatically

### Caching Strategy

**Redis Implementation** (`apps/backend/src/redis/redis.service.ts`):

**Cache Keys:**
- `users` - List of all users (TTL: 5 seconds)
- `user:{id}` - Individual user (TTL: 120 seconds)

**Cache Invalidation:**
```typescript
// On update or delete
await this.redisService.del(`user:${id}`);
await this.redisService.del('users');
```

**Pattern:**
1. Check cache first
2. If miss, fetch from database
3. Store in cache before returning
4. Invalidate on mutations

## Testing Strategy

### Backend Testing

**Unit Tests** (`*.spec.ts`):
```typescript
// Mock dependencies
const mockPrisma = {
  user: {
    findMany: jest.fn(),
    create: jest.fn(),
  }
};

const mockRedis = {
  get: jest.fn(),
  set: jest.fn(),
  del: jest.fn(),
};

// Create testing module
const module = await Test.createTestingModule({
  providers: [
    UsersService,
    { provide: PrismaService, useValue: mockPrisma },
    { provide: RedisService, useValue: mockRedis },
  ],
}).compile();
```

**E2E Tests** (`test/*.e2e-spec.ts`):
```typescript
// Full application bootstrap
const moduleFixture = await Test.createTestingModule({
  imports: [AppModule],
}).compile();

app = moduleFixture.createNestApplication();
await app.init();

// Use supertest for HTTP requests
const loginResponse = await request(app.getHttpServer())
  .post('/auth/login')
  .send({ email, password });

const cookie = loginResponse.headers['set-cookie'];

await request(app.getHttpServer())
  .get('/users')
  .set('Cookie', cookie)
  .expect(200);
```

**Run Tests:**
```bash
# Unit tests
pnpm --filter backend test

# E2E tests (requires Docker)
pnpm --filter backend test:e2e

# Coverage
pnpm --filter backend test:cov
```

### Frontend Testing

**Component Tests** (`*.test.tsx`):
```typescript
// Mock service layer
jest.mock('@/lib/services/userService');

// Create test query client
const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
  },
});

// Render with providers
render(
  <QueryClientProvider client={createTestQueryClient()}>
    <UserList />
  </QueryClientProvider>
);

// Use React Testing Library
await waitFor(() => {
  expect(screen.getByText('Alice')).toBeInTheDocument();
});
```

**Run Tests:**
```bash
# Unit tests
pnpm --filter web test

# With coverage
pnpm --filter web test:coverage
```

### Test Coverage

Tests are co-located with source files for easy maintenance:
```
users/
├── services/
│   ├── users.service.ts
│   └── users.service.spec.ts
└── components/
    ├── UserList.tsx
    └── UserList.test.tsx
```

## Database Schema

**Location:** `apps/backend/prisma/schema.prisma`

**Current Models:**

```prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String   # bcrypt hashed
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**Seeded Users** (via `prisma/seed.ts`):
- Alice (alice@example.com / password123)
- Bob (bob@example.com / password123)

### Making Schema Changes

1. Edit `schema.prisma`
2. Run migration:
   ```bash
   pnpm --filter backend prisma:migrate:docker
   ```
3. Generate client:
   ```bash
   pnpm --filter backend prisma:generate:docker
   pnpm --filter backend prisma:generate:local  # For VSCode
   ```
4. Update seed if needed
5. Restart backend

## Common Tasks

### Adding a New Feature

**Backend:**

1. Create feature directory: `src/feature-name/`
2. Add controller: `feature-name/controllers/feature.controller.ts`
3. Add service: `feature-name/services/feature.service.ts`
4. Add DTOs: `feature-name/dto/create-feature.dto.ts`
5. Create module: `feature-name/feature.module.ts`
6. Import module in `app.module.ts`
7. Add tests: `*.spec.ts` files co-located
8. Add E2E tests: `test/feature.e2e-spec.ts`

**Frontend:**

1. Create feature directory: `app/feature-name/`
2. Add page: `feature-name/page.tsx`
3. Add components: `feature-name/components/Component.tsx`
4. Add service: `lib/services/featureService.ts`
5. Add tests: `*.test.tsx` files co-located
6. Update middleware if route needs protection

### Adding a Database Model

1. Edit `apps/backend/prisma/schema.prisma`
2. Create migration:
   ```bash
   pnpm --filter backend prisma:migrate:docker
   ```
3. Generate client:
   ```bash
   pnpm --filter backend prisma:generate:docker
   pnpm --filter backend prisma:generate:local
   ```
4. Update seed file if needed
5. Create NestJS module/service for the model
6. Add tests

### Adding a Shared Component

1. Create in `packages/ui/src/component-name.tsx`
2. Export from `packages/ui/src/index.tsx`
3. Import in apps: `import { Component } from '@repo/ui'`
4. Component generator available:
   ```bash
   cd packages/ui
   pnpm turbo gen react-component
   ```

### Debugging

**Backend:**
```bash
# View backend logs
docker-compose logs -f backend

# Access backend container
docker-compose exec backend sh

# Check database
docker-compose exec postgres psql -U postgres -d app

# Check Redis
docker-compose exec redis redis-cli
```

**Frontend:**
```bash
# View frontend logs
docker-compose logs -f web

# Access frontend container
docker-compose exec web sh
```

**General:**
```bash
# Check all services status
docker-compose ps

# Restart service
docker-compose restart backend

# Rebuild service
docker-compose up -d --build backend
```

## Important Considerations

### 1. Prisma Client Generation

**ALWAYS generate Prisma client in both contexts after schema changes:**

```bash
# For Docker (runtime)
pnpm --filter backend prisma:generate:docker

# For local development (VSCode IntelliSense)
pnpm --filter backend prisma:generate:local
```

Forgetting the local generation will cause TypeScript errors in your editor.

### 2. Environment Variables

**Different environments require different variables:**

- Docker uses service names (`postgres`, `redis`)
- Local uses `localhost`
- Backend has two ports: internal `3000`, external `3001`
- Frontend API URL differs for SSR vs client-side

### 3. Authentication Cookies

**HTTP-only cookies require special handling:**

- Frontend API routes proxy auth requests
- All fetch calls must include `credentials: 'include'`
- CORS configured for `localhost:3000` in backend
- Middleware checks `token` cookie for route protection

### 4. Redis Caching

**Cache invalidation is critical:**

```typescript
// Always invalidate both specific and list caches
await this.redisService.del(`user:${id}`);
await this.redisService.del('users');
```

### 5. Feature-Based Architecture

**Keep features self-contained:**

- All related code in one directory
- Tests co-located with source
- Minimize cross-feature dependencies
- Clear public API through exports

### 6. Testing Requirements

**Tests must pass before committing:**

```bash
# Backend
pnpm --filter backend test
pnpm --filter backend test:e2e

# Frontend
pnpm --filter web test

# All
pnpm test
```

### 7. Type Safety

**Strict TypeScript is enforced:**

```bash
# Check types before committing
pnpm check-types
```

### 8. Code Formatting

**Prettier runs automatically but can be invoked:**

```bash
pnpm format
```

### 9. Turborepo Caching

Turborepo caches build outputs. If you encounter stale builds:

```bash
# Clear Turborepo cache
rm -rf .turbo

# Clear all node_modules
rm -rf node_modules apps/*/node_modules packages/*/node_modules

# Reinstall
pnpm install
```

### 10. Docker Resource Limits

Services have memory limits:
- postgres: 256MB
- redis: 128MB
- backend: 384MB
- web: 768MB

Monitor with: `docker stats`

## Key Files Reference

### Configuration Files

```
/home/user/take-home/turbo.json              # Turborepo config
/home/user/take-home/docker-compose.yml      # Docker orchestration
/home/user/take-home/pnpm-workspace.yaml     # pnpm workspaces

# Backend
/home/user/take-home/apps/backend/prisma/schema.prisma
/home/user/take-home/apps/backend/src/main.ts
/home/user/take-home/apps/backend/src/app.module.ts
/home/user/take-home/apps/backend/init.sh
/home/user/take-home/apps/backend/Dockerfile

# Frontend
/home/user/take-home/apps/web/app/layout.tsx
/home/user/take-home/apps/web/app/providers.tsx
/home/user/take-home/apps/web/middleware.ts
/home/user/take-home/apps/web/lib/queryClient.ts
/home/user/take-home/apps/web/Dockerfile
```

### Entry Points

```
Backend:  apps/backend/src/main.ts          (port 3000 internal, 3001 external)
Frontend: apps/web/app/page.tsx             (redirects to /auth/register)
          apps/web/app/layout.tsx           (root layout with providers)
```

### Key Modules

```
Backend:
  - apps/backend/src/auth/           # Authentication (JWT)
  - apps/backend/src/users/          # Users CRUD
  - apps/backend/src/prisma/         # Database client
  - apps/backend/src/redis/          # Cache client

Frontend:
  - apps/web/app/auth/               # Auth pages
  - apps/web/app/users/              # Users pages
  - apps/web/lib/services/           # API client
```

## Troubleshooting

### "Prisma Client Not Found"

```bash
pnpm --filter backend prisma:generate:docker
pnpm --filter backend prisma:generate:local
```

### "Cannot Connect to Database"

Check environment variables match running services:
```bash
# For Docker
DATABASE_URL=postgres://postgres:postgres@postgres:5432/app

# For local
DATABASE_URL=postgres://postgres:postgres@localhost:5432/app
```

### "401 Unauthorized on /users"

Ensure you're logged in and cookie is being sent:
```typescript
fetch(url, { credentials: 'include' })  // Required
```

### "Port Already in Use"

```bash
# Find process
lsof -i :3000  # or :3001

# Stop Docker services
docker-compose down
```

### "Type Errors in VSCode"

```bash
# Regenerate Prisma client locally
pnpm --filter backend prisma:generate:local

# Restart TypeScript server in VSCode
# Cmd+Shift+P > "TypeScript: Restart TS Server"
```

### "Tests Failing After Schema Change"

1. Run migrations: `pnpm --filter backend prisma:migrate:docker`
2. Generate client: `pnpm --filter backend prisma:generate:docker`
3. Seed database: `pnpm --filter backend prisma:seed`
4. Update test mocks to match new schema

### "Docker Build Fails"

```bash
# Clean rebuild
docker-compose down -v
docker-compose up -d --build

# Clear Docker cache
docker system prune -a
```

## Additional Resources

- **Project README:** `/home/user/take-home/README.md`
- **Backend README:** `/home/user/take-home/apps/backend/README.md` (if exists)
- **Frontend README:** `/home/user/take-home/apps/web/README.md` (if exists)
- **Turborepo Docs:** https://turbo.build/repo/docs
- **NestJS Docs:** https://docs.nestjs.com/
- **Next.js 15 Docs:** https://nextjs.org/docs
- **Prisma Docs:** https://www.prisma.io/docs
- **React Query Docs:** https://tanstack.com/query/latest

---

**Last Updated:** 2025-11-23
**Monorepo Tool:** Turborepo 2.5.0
**Package Manager:** pnpm 9.0.0
**Node Version:** 18+

---
> Source: [roxdavirox/take-home](https://github.com/roxdavirox/take-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
