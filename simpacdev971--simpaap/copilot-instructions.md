## simpaap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Simpaap is a multi-tenant SaaS application built with Next.js 16 (App Router). Each tenant gets their own subdomain (e.g., `tenant.print.simp.ac`) with isolated data and customizable applications.

## Commands

```bash
pnpm dev              # Development server
pnpm build            # Production build (runs prisma generate first)
pnpm lint             # ESLint + TypeScript checks
pnpm dev:proxy        # Dev server with proxy for subdomain testing
pnpm prisma db push   # Sync Prisma schema to database
pnpm prisma generate  # Generate Prisma client
```

## Architecture

### Multi-Tenancy via Middleware

The core routing logic is in `middleware.ts`:
- Extracts subdomain from hostname and rewrites to `/[subdomain]/*` routes
- Uses LRU cache (5 min TTL, 500 max entries) to avoid repeated DB lookups for tenant validation
- Reserved subdomains (`www`, `app`, `dashboard`) are handled separately from tenant subdomains
- Invalid subdomains return 404; accessing tenant routes via path (e.g., `/tenant/page`) is blocked

### Authentication & Authorization

- **NextAuth** with JWT strategy and Prisma adapter (`app/api/auth/[...nextauth]/route.ts`)
- Credentials provider with bcrypt password hashing
- Session includes: `id`, `email`, `role`, `tenantSlug`, `theme`
- Three roles: `SUPERADMIN`, `ADMIN`, `MEMBER`

**Route Protection:**
- `lib/auth/permissions.ts` - Centralized route permission config
- `components/auth/ProtectedPage.tsx` - Server-side page protection wrapper
- `components/auth/ProtectedComponent.tsx` - Client-side component protection
- `requireTenantMatch: true` ensures ADMINs can only access their own tenant

### Database

PostgreSQL via Prisma ORM. Key models in `prisma/schema.prisma`:
- **Tenant** - Organization with unique subdomain
- **User** - Belongs to one tenant, has role
- **Application** / **tenant_application** - Apps assigned to tenants
- **categorie** - Application categories

### File Structure

```
app/
├── [subdomain]/          # Tenant-specific routes (login, admin, simpaap tools)
├── api/                  # API routes (tenant CRUD, auth, user management)
└── page.tsx              # Landing page

components/
├── auth/                 # ProtectedPage, withAuth HOC
├── admin/                # TenantsCrud, UsersCrud
├── app/print/            # PDF viewer and print-related components
└── ui/                   # shadcn/ui components

lib/
├── auth/                 # permissions.ts, getUserAccess.ts, utils.ts
├── prisma.ts             # Singleton Prisma client
└── types/auth.ts         # TypeScript types for auth system
```

### Key API Endpoints

- `GET /api/tenant/list` - Returns all tenant subdomains (used by middleware cache)
- `GET /api/tenant?subdomain=x` - Check if tenant exists
- `POST /api/create-tenant` - Create new tenant
- `DELETE /api/tenant/delete` - Delete tenant
- `/api/tenant/users` - User management within tenant

## Environment Variables

Required in `.env`:
```
DATABASE_URL=           # PostgreSQL connection string
DIRECT_URL=             # Direct DB URL (for Prisma migrations)
NEXTAUTH_URL=           # Base URL for auth callbacks
NEXTAUTH_SECRET=        # JWT signing secret
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

## Tech Stack

- Next.js 16 with App Router
- Prisma ORM + PostgreSQL
- NextAuth.js for authentication
- Tailwind CSS + shadcn/ui
- Zustand for state management
- Framer Motion for animations

---
> Source: [SimpacDev971/Simpaap](https://github.com/SimpacDev971/Simpaap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
