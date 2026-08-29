## amazon

> Full-stack Amazon clone e-commerce web application with product catalog, cart, checkout, orders, user accounts, reviews/ratings, search, wishlist, seller dashboard, and recommendations.

# Amazon Clone — Project Guide

## Project Overview

Full-stack Amazon clone e-commerce web application with product catalog, cart, checkout, orders, user accounts, reviews/ratings, search, wishlist, seller dashboard, and recommendations.

## Tech Stack

| Layer          | Technology                          |
| -------------- | ----------------------------------- |
| Frontend       | React 18+ with TypeScript           |
| Styling        | Tailwind CSS                        |
| State Mgmt     | Zustand (client state) + TanStack Query (server state) |
| Backend        | Node.js with Fastify                |
| Database       | PostgreSQL                          |
| ORM            | Prisma                              |
| Auth           | JWT (access + refresh token rotation) |
| Testing        | Vitest + React Testing Library + Playwright |
| Package Mgr    | pnpm (with workspaces)              |
| Validation     | Zod                                 |
| Build Tool     | Vite (frontend)                     |

## Project Structure (Monorepo)

```
amazon/
├── CLAUDE.md
├── pnpm-workspace.yaml
├── package.json              # Root scripts, shared devDependencies
├── tsconfig.base.json        # Shared TypeScript config
├── .env.example
├── .gitignore
├── docs/
│   ├── api-routes.md         # Full API route reference
│   └── db-schema.md          # Database schema reference
├── packages/
│   └── shared/               # Shared types, constants, validation schemas
│       ├── src/
│       │   ├── types/        # Shared TypeScript interfaces/types
│       │   ├── constants/    # Shared constants (status codes, enums)
│       │   └── validators/   # Zod schemas shared between client & server
│       ├── package.json
│       └── tsconfig.json
├── client/                   # React frontend
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/       # Reusable UI components
│   │   │   ├── ui/           # Primitives (Button, Input, Modal, etc.)
│   │   │   ├── layout/       # Header, Footer, Sidebar, etc.
│   │   │   └── features/     # Feature-specific components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── pages/            # Route-level page components
│   │   ├── stores/           # Zustand stores
│   │   ├── services/         # API client functions (fetch/axios wrappers)
│   │   ├── utils/            # Frontend utility functions
│   │   ├── types/            # Frontend-only types
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── router.tsx        # React Router config
│   ├── e2e/                  # Playwright E2E tests
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   ├── postcss.config.js
│   ├── tsconfig.json
│   └── package.json
├── server/                   # Fastify backend
│   ├── src/
│   │   ├── plugins/          # Fastify plugins (auth, cors, etc.)
│   │   ├── routes/           # Route handlers grouped by domain
│   │   │   ├── auth/
│   │   │   ├── products/
│   │   │   ├── cart/
│   │   │   ├── orders/
│   │   │   ├── reviews/
│   │   │   ├── users/
│   │   │   ├── wishlist/
│   │   │   ├── sellers/
│   │   │   └── search/
│   │   ├── services/         # Business logic layer
│   │   ├── middlewares/       # Auth guards, validation, rate limiting
│   │   ├── utils/            # Server utility functions
│   │   ├── types/            # Server-only types
│   │   ├── config/           # Environment config, constants
│   │   └── app.ts            # Fastify app setup
│   ├── prisma/
│   │   ├── schema.prisma
│   │   ├── migrations/
│   │   └── seed.ts           # Database seeding
│   ├── tests/                # Vitest unit/integration tests
│   ├── tsconfig.json
│   └── package.json
└── docker-compose.yml        # PostgreSQL + Redis (optional) for local dev
```

## Coding Conventions

### General

- **Language**: TypeScript everywhere — strict mode enabled, no `any` types
- **Formatting**: Use Prettier (default config) + ESLint
- **Imports**: Use path aliases (`@/components`, `@/hooks`, `@/services`, etc.)
- **Naming**:
  - Files/folders: `kebab-case` (e.g., `product-card.tsx`, `auth-service.ts`)
  - Components: `PascalCase` (e.g., `ProductCard`, `CartDrawer`)
  - Functions/variables: `camelCase`
  - Types/Interfaces: `PascalCase`, no `I` prefix (e.g., `User`, not `IUser`)
  - Constants: `UPPER_SNAKE_CASE`
  - Database tables: `snake_case` (Prisma maps to PascalCase models)
- **Exports**: Named exports only — no default exports (except pages)

### Frontend (React)

- Functional components only with arrow functions
- Props type defined inline or with a `Props` suffix (e.g., `ProductCardProps`)
- Use Zustand stores for global client state (cart, UI, auth user)
- Use TanStack Query for all server data fetching and caching
- Tailwind CSS for all styling — no CSS modules or styled-components
- Component structure: UI primitives in `components/ui/`, composed into feature components
- React Router v6+ with lazy loading for route-level code splitting
- **Responsive design**: Mobile-first approach — design for small screens first, scale up with Tailwind breakpoints (`sm:`, `md:`, `lg:`, `xl:`)
- **Accessibility**: Use semantic HTML elements (`nav`, `main`, `button`, etc.), add ARIA attributes where needed, ensure full keyboard navigation support, maintain sufficient color contrast ratios

### Design Reference

- Use screenshots in `docs/inspiration/` as the primary visual reference for UI work; when no relevant reference exists, follow established e-commerce UX patterns

### Backend (Fastify)

- Route handlers are thin — delegate logic to service layer
- Services contain business logic and call Prisma for data access
- Use Fastify's built-in schema validation (JSON Schema / Zod via fastify-type-provider-zod)
- Consistent error responses: `{ success: false, error: { code, message } }`
- Success responses: `{ success: true, data: ... }`
- All routes prefixed with `/api/v1/`
- Use Fastify plugins for cross-cutting concerns (auth, CORS, rate limiting)

### Database (Prisma)

- All models include `id`, `createdAt`, `updatedAt` fields
- Use UUID for primary keys
- Define relations explicitly in schema
- Write seed data for development (`prisma/seed.ts`)
- Create migrations with descriptive names: `pnpm prisma migrate dev --name add-wishlist-table`

### Authentication

- Access tokens: short-lived (15 min), stored in memory (Zustand)
- Refresh tokens: long-lived (7 days), stored in httpOnly secure cookie
- Token rotation: issue new refresh token on each refresh
- Protected routes use a Fastify `preHandler` hook for JWT verification

## Key Commands

```bash
# Install dependencies
pnpm install

# Development
pnpm dev                  # Run both client and server concurrently
pnpm dev:client           # React dev server (Vite)
pnpm dev:server           # Fastify dev server (tsx watch)

# Database
pnpm db:migrate           # Run Prisma migrations
pnpm db:seed              # Seed the database
pnpm db:studio            # Open Prisma Studio
pnpm db:reset             # Reset and re-seed database

# Testing
pnpm test                 # Run all Vitest tests
pnpm test:client          # Frontend tests only
pnpm test:server          # Backend tests only
pnpm test:e2e             # Playwright E2E tests

# Build
pnpm build                # Build both client and server
pnpm build:client         # Build React app
pnpm build:server         # Build Fastify server

# Linting & Formatting
pnpm lint                 # ESLint check
pnpm format               # Prettier format
pnpm typecheck            # TypeScript type checking
```

## Environment Variables

Required in `.env` (never commit this file):

```
DATABASE_URL=postgresql://user:password@localhost:5432/amazon_clone
JWT_ACCESS_SECRET=<random-secret>
JWT_REFRESH_SECRET=<random-secret>
PORT=3001
CLIENT_URL=http://localhost:5173
NODE_ENV=development
```

## Git Workflow

- **Branch naming**: `feature/<short-description>`, `fix/<short-description>`, `chore/<short-description>` (e.g., `feature/add-wishlist`, `fix/cart-total-calculation`)
- **Commit messages**: Use conventional commits format:
  - `feat: add wishlist page`
  - `fix: correct cart total when item removed`
  - `refactor: extract product card component`
  - `test: add order service unit tests`
  - `chore: update dependencies`
  - `docs: add search endpoint to api-routes`
- **Commit splitting**: Split changes into multiple logical commits when appropriate (e.g., separate commits for schema changes, business logic, and UI work)
- **PR process**: One feature/fix per PR, squash merge into `main`
- **Base branch**: `main` is the default branch — all PRs target `main`

## API Design

- Follow RESTful conventions, prefix all routes with `/api/v1/`
- See `docs/api-routes.md` for the full route reference

## Important Rules for AI Assistant

### Approval Required

- **Deletions**: Always ask the user for confirmation before deleting any file, folder, or code block
- **Database actions**: All database-related actions require explicit user approval first (migrations, schema changes, seeding, resetting, dropping tables, modifying data)
- **Git actions**: All git operations require explicit user approval first (commits, pushes, branch creation/deletion, merges, rebases, stashing)
- **CLAUDE.md updates**: Keep CLAUDE.md in sync with the project. Proactively suggest updates whenever tech stack, project structure, conventions, workflows, or rules change. Also flag any outdated or incorrect information. Never modify without explicit user approval first.
- **Docs updates**: Keep `docs/api-routes.md` in sync with the API. Proactively suggest updates whenever API endpoints are created, changed, or removed. Also flag any outdated or incorrect information. Never modify without explicit user approval first.
- **Schema docs**: Keep `docs/db-schema.md` in sync with the Prisma schema. Proactively suggest updates whenever models, enums, or relations are added, changed, or removed. Also flag any outdated or incorrect information. Never modify without explicit user approval first.

### Communication

- Only ask the user questions on critical issues — avoid unnecessary interruptions for trivial decisions
- When in doubt about destructive or irreversible actions, always ask

### Code Quality

1. Always use TypeScript — never write plain JavaScript files
2. Never use `any` type — use `unknown` and narrow, or define proper types
3. Never store secrets or credentials in code — always use environment variables
4. Always validate user input on both client and server
5. Use Prisma transactions for multi-step database operations
6. Keep components small and focused — extract when exceeding ~100 lines
7. Write tests for business logic in services and critical user flows
8. Use proper HTTP status codes in API responses
9. Handle loading and error states in all data-fetching components
10. Use Tailwind's design system — avoid arbitrary values when a utility exists

---
> Source: [paula-a-pham/amazon](https://github.com/paula-a-pham/amazon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
