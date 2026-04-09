## portaldalembranca

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Memorial QR MVP (Portal da Lembrança) is a full-stack memorial management platform for funeral homes, families, and public visitors. The system enables funeral homes to create memorial pages with QR codes, families to manage content, and visitors to leave dedications.

## Development Commands

### Running the Application
- `npm run dev` - Start API server in watch mode (development)
- `npm run build` - Lint and compile server TypeScript to `dist/`
- `npm run build:client` - Build client-side static assets with Vite to `dist/public`
- `npm run build:vercel` - Production build for Vercel deployment (lint + client build)
- `npm start` - Run compiled production server

### Testing
- `npm test` - Run all tests with Vitest
- `npm run test:cov` - Run tests with coverage report

### Code Quality
- `npm run lint` - Type-check server code and run ESLint (compiles to temp directory)
- `npm run format` - Format all code with Prettier

### Database Operations
- `npm run db:generate` - Generate Drizzle migrations from schema changes
- `npm run db:migrate` - Apply migrations (local development)
- `npm run db:push` - Push schema directly to database (development)
- `npm run db:migrate:neon` - Apply migrations to Neon database
- `npm run db:studio` - Open Drizzle Studio for database inspection
- `npm run db:seed:historical` - Seed database with historical memorial data

## Architecture

### Dual Server Architecture

The project uses a **hybrid architecture** combining two different backend approaches:

1. **NestJS-like API** (`server/api/`): Custom lightweight framework inspired by NestJS using decorators
   - Located in `server/api/src/`
   - Uses decorators (`@Controller`, `@Get`, `@Post`, `@Injectable`)
   - Main entry point: `server/api/src/main.ts`
   - Custom framework implementation: `server/api/src/framework/nest-like.ts`
   - Currently handles: Payment processing (Stripe integration)
   - Runs on Express with decorator-based routing

2. **tRPC API** (`server/`): Type-safe RPC layer for main application logic
   - Main router: `server/routers.ts`
   - Core utilities: `server/_core/` (trpc setup, context, cookies, SDK integration)
   - End-to-end type safety with client through TypeScript
   - Procedure types: `publicProcedure`, `protectedProcedure`, `adminProcedure`
   - Uses SuperJSON for serialization (supports Date objects, Maps, Sets)

Both servers run simultaneously in development. The NestJS-like API handles REST endpoints at `/api/*` while tRPC handles type-safe procedures at `/api/trpc`.

### Database Layer

- **ORM**: Drizzle with PostgreSQL (Neon/Vercel Postgres)
- **Schema**: `drizzle/schema.ts` - single source of truth
- **Database helpers**: `server/db.ts` - query functions abstraction
- **Connection**: Lazy initialization, single connection pooling

### Frontend

- **Framework**: React 19 + TypeScript
- **Routing**: Wouter (lightweight client-side routing)
- **Styling**: Tailwind CSS 4 + shadcn/ui components
- **State Management**: tRPC React Query hooks
- **Build**: Vite with TypeScript path aliases (`@/*` for client, `@shared/*` for shared)

### Data Flow

**Authentication Flow:**
1. Frontend calls `trpc.auth.funeralHomeLogin` or `trpc.auth.familyUserLogin`
2. Backend validates with bcrypt, creates JWT session token
3. Session stored in HTTP-only cookie via `persistUserSession`
4. Subsequent requests include cookie automatically
5. tRPC middleware (`requireUser`) validates session in protected procedures

**Authorization Hierarchy:**
- `publicProcedure` - No authentication required
- `protectedProcedure` - Requires authenticated user (funeral home or family)
- `adminProcedure` - Requires admin role

**OpenID Prefixes:**
- Funeral homes: `funeral-{id}`
- Family users: `family-{id}`
- System identifies user type by openId prefix

### Key Architectural Patterns

**Slug Generation**: Memorials use unique slugs combining name, timestamp, and random string (`generateSlug` in `server/routers.ts:44`)

**Invitation Flow**: Family users receive invitation tokens (7-day expiry) when memorials are created. They must accept invitation to set password and activate account.

**QR Code System**: Each memorial can generate QR codes (PNG/SVG) linking to public memorial page via `server/qrcode.ts`

**Database Abstraction**: All database operations go through helper functions in `server/db.ts`, never direct Drizzle calls in routers. This enables testing, caching, and consistent error handling.

### User Roles & Permissions

**Three user types:**
1. **Funeral Homes** (`funeralHomes` table): Create memorials, manage their own content
2. **Family Users** (`familyUsers` table): Edit assigned memorials, upload photos, manage content
3. **Admin Users** (`adminUsers` table): View all data, manage leads, production queue

**Authorization Rules:**
- Funeral homes: Only access memorials where `funeralHomeId` matches their ID
- Family users: Only access memorials where `familyUserId` matches their ID
- Admins: Full access to all resources
- Public visitors: Read-only access to memorials with `visibility: "public"` and `status: "active"`

### Admin Module

Production queue system for tracking memorial creation:
- **Orders** (`orders` table): Track memorial production status
- **Order History** (`orderHistory` table): Audit trail of status changes
- **Statuses**: new → in_production → waiting_data → ready → delivered (or cancelled)
- **Priorities**: low, normal, high, urgent

## Environment Configuration

See `.env.example` for all variables. Key configurations:

**Database:**
- `DATABASE_URL` - PostgreSQL connection string (required for all DB operations)

**Authentication:**
- `JWT_SECRET` - Session token signing key
- `OAUTH_SERVER_URL` - External OAuth provider (if using OAuth)
- `OWNER_OPEN_ID` - Owner's openId for auto-admin assignment

**Payments:**
- `STRIPE_SECRET_KEY` - Backend Stripe key (optional - returns mock if unset)
- `VITE_STRIPE_PUBLISHABLE_KEY` - Frontend Stripe key

**Frontend:**
- `VITE_APP_TITLE` - Application title
- `VITE_FRONTEND_FORGE_API_KEY` - For map features

## Testing

Tests use Vitest with Node environment. Test files located in `server/**/*.spec.ts` or `server/**/*.test.ts`.

Example test structure (see `server/api/src/app.controller.spec.ts`):
```typescript
import { describe, expect, it, beforeEach } from 'vitest';
import { TestingFactory } from './framework/nest-like';

describe('TestCase', () => {
  beforeEach(async () => {
    const moduleRef = await TestingFactory.createTestingModule({
      controllers: [MyController],
      providers: [MyService],
    }).compile();
    // ...
  });
});
```

## Database Schema

Core entities (see `drizzle/schema.ts` for full schema):

- **users** - OAuth users (mainly for SDK integration)
- **funeralHomes** - Funeral home accounts with email/password auth
- **familyUsers** - Family member accounts with invitation flow
- **memorials** - Core memorial records with slug-based public URLs
- **descendants** - Children/grandchildren relationships
- **photos** - Memorial photo galleries (ordered)
- **dedications** - Public tribute messages from visitors
- **leads** - Contact requests from landing page
- **orders** - Production queue for memorial creation
- **adminUsers** - System administrators

**Important relationships:**
- Memorial ← funeralHomeId (who created it)
- Memorial ← familyUserId (who manages it)
- Photos, Descendants, Dedications → memorialId (belongs to memorial)
- Orders → memorialId, funeralHomeId, familyUserId

## Common Development Patterns

**Adding a new tRPC procedure:**
1. Define input schema with Zod
2. Add database helper function to `server/db.ts` if needed
3. Create procedure in appropriate router in `server/routers.ts`
4. Frontend automatically gets types via `AppRouter` export

**Adding a database field:**
1. Update schema in `drizzle/schema.ts`
2. Run `npm run db:generate` to create migration
3. Run `npm run db:push` or `npm run db:migrate` to apply
4. Update TypeScript types are automatically inferred by Drizzle

**Adding a new page:**
1. Create component in `client/src/pages/`
2. Add route in `client/src/App.tsx`
3. Use tRPC hooks (`trpc.*.useQuery()` or `trpc.*.useMutation()`)

## Deployment

**Vercel (Current):**
- Uses `vercel.json` configuration
- Build command: `npm run build:vercel`
- Output: `dist/public` (static client files)
- Framework: Vite (SPA mode)
- Rewrites handle client-side routing

**AWS (via Terraform):**
- Infrastructure defined in `infra/terraform/`
- Uses Docker (`Dockerfile` in root)
- ECS Fargate + ALB + ECR setup
- See README.md for deployment steps

## Package Manager

Project uses **pnpm** (specified in `package.json` engines). Note:
- Patched dependency: `wouter@3.7.1` (see `patches/wouter@3.7.1.patch`)
- Override: `tailwindcss>nanoid` pinned to 3.3.7

## Important Notes

- **Never use `git` commands with `-uall` flag** - can cause memory issues on large repos
- **Database connection is lazy** - code can run without DATABASE_URL for tooling purposes
- **Stripe integration degrades gracefully** - returns mock payment if STRIPE_SECRET_KEY not set
- **Two TypeScript configs**: `tsconfig.json` (client) and `tsconfig.server.json` (server API)
- **Server uses CommonJS** - `type: "commonjs"` in package.json, required for decorator support
- **Client uses ESNext** - Modern bundler target for Vite

## File Structure Highlights

```
├── server/
│   ├── _core/          # Core utilities (tRPC, SDK, context, cookies)
│   ├── api/src/        # NestJS-like API (decorators, Express routes)
│   ├── routers.ts      # Main tRPC router definitions
│   ├── db.ts           # Database query helpers
│   └── qrcode.ts       # QR code generation
├── client/src/
│   ├── pages/          # Route components
│   ├── components/     # Reusable UI components
│   ├── lib/            # Client utilities (tRPC client)
│   └── hooks/          # Custom React hooks
├── drizzle/
│   ├── schema.ts       # Database schema (source of truth)
│   └── migrations/     # Version-controlled SQL migrations
├── shared/             # Code shared between client and server
└── scripts/            # Utility scripts (migrations, seeding)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielmedeiros52)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielmedeiros52)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
