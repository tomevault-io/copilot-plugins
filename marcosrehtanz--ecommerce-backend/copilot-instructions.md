## ecommerce-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dynnamo is a full-featured B2C e-commerce platform with a NestJS backend and Next.js frontend. The project significantly exceeds MVP requirements with 11 modules, 52+ API endpoints, payment integration, analytics dashboard, and comprehensive documentation.

**Project Status:** MVP+ Complete (Production-ready with minor improvements needed)

## Key Features

- **Authentication:** JWT with access + refresh tokens, automatic rotation
- **Payments:** MercadoPago Checkout Pro with webhooks
- **Dashboard:** Real-time analytics with charts and CSV export
- **Email:** Transactional emails with Handlebars templates
- **Security:** Rate limiting (3 levels), Helmet, JWT exception filter

## Commands

### Backend (from `backend/` directory)

```bash
# Development
npm run start:dev          # Start with hot reload

# Build
npm run build              # Compile TypeScript

# Database
npx prisma migrate dev     # Run migrations
npx prisma db seed         # Seed initial data (admin@example.com / Admin123!)
npx prisma studio          # Visual database browser
npx prisma generate        # Regenerate Prisma client

# Testing
npm test                   # Run unit tests
npm run test:watch         # Watch mode
npm run test:e2e           # End-to-end tests

# Linting
npm run lint               # ESLint with autofix
npm run format             # Prettier formatting
```

### Frontend (from `frontend/` directory)

```bash
npm run dev                # Start dev server on port 3001
npm run build              # Production build
npm run lint               # Next.js linting
```

### Docker (from `backend/` directory)

```bash
# Development (database only)
docker-compose -f docker-compose.dev.yml up -d

# Production (backend + database)
docker-compose up -d
```

## Architecture

### Backend Structure

The backend follows NestJS modular architecture with **global guards and filters**:

**Global Guards (app.module.ts):**
- `ThrottlerGuard` - Rate limiting (3 levels: short, medium, long)
- `JwtAuthGuard` - All routes require authentication by default
- `RolesGuard` - Role-based access control

**Global Filter:**
- `JwtExceptionFilter` - Returns error codes (TOKEN_EXPIRED, TOKEN_INVALID, etc.)

**Global Interceptor:**
- `LoggingInterceptor` - Request logging

**Custom Decorators:**
- `@Public()` - Make endpoints public (skip auth)
- `@Roles('ADMIN')` - Restrict to admin-only
- `@CurrentUser()` - Get the authenticated user

**Modules (11 total):**
- `auth` - JWT authentication with refresh tokens
- `users` - User management (Admin CRUD)
- `products` - Product catalog with best sellers
- `categories` - Hierarchical categories
- `cart` - Shopping cart with local/server sync
- `orders` - Order management with state machine
- `payments` - MercadoPago integration with webhooks
- `reports` - Dashboard analytics and CSV export
- `notifications` - Email system with templates
- `newsletter` - Email subscriptions
- `site-config` - Dynamic configuration

Each module follows the pattern: `module.ts`, `controller.ts`, `service.ts`, `dto/`

### Frontend Structure

- **State Management**: Zustand for local state (auth, cart), TanStack Query for server state
- **UI Components**: Mantine UI v7.6 + Recharts for charts
- **API Layer**: Axios instance at `lib/api/axios.ts` with interceptors for auth tokens and automatic refresh
- **Route Groups**: `(auth)` for login/register, `(shop)` for customer pages, `admin/` for admin panel
- **Custom Hooks**: 12 hooks encapsulating all API logic

### Key Patterns

**Cart Sync Strategy**: Anonymous users have local cart (Zustand). On login, local cart merges with server cart using MAX quantity strategy.

**Order Status Flow**: PENDING → CONFIRMED → SHIPPED → DELIVERED (or CANCELLED at PENDING/CONFIRMED)

**Payment Flow**: Create preference → Redirect to MercadoPago → Webhook updates order

**Token Refresh**: Axios interceptor detects 401, queues requests, refreshes token, retries all

**Order Limits**: Only 1 pending unpaid order per user allowed

## Database

PostgreSQL with Prisma ORM. Schema at `backend/prisma/schema.prisma`.

**Models:** User, Product, Category, Cart, CartItem, Order, OrderItem, SiteConfig, NewsletterSubscription

**Enums:**
- `Role` (ADMIN, USER)
- `OrderStatus` (PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED)
- `PaymentStatus` (PENDING, APPROVED, REJECTED, CANCELLED)

## API URLs

- Backend: http://localhost:3000
- Frontend: http://localhost:3001
- Swagger: http://localhost:3000/api/docs

## Test Users

| Email | Password | Role |
|-------|----------|------|
| admin@example.com | Admin123! | ADMIN |
| user@example.com | User123! | USER |

## Test Credit Cards (MercadoPago)

| Number | CVV | Expiry | Name | Result |
|--------|-----|--------|------|--------|
| 5031 7557 3453 0604 | 123 | 11/30 | APRO | Approved |
| 5031 7557 3453 0604 | 123 | 11/30 | OTHE | Rejected |

## Documentation

| File | Description |
|------|-------------|
| `docs/README.md` | Main project documentation |
| `docs/ROADMAP.md` | Evolution plan with 6 phases |
| `docs/EVALUACION_TECNICA.md` | Technical evaluation (10/10) |
| `docs/security.md` | Security implementations |
| `docs/payments-module.md` | MercadoPago integration |
| `docs/cart-module.md` | Cart sync logic and truth table |
| `docs/orders-module.md` | Order states and transitions |
| `docs/reports-module.md` | Dashboard and analytics |
| `docs/notifications-module.md` | Email templates and SMTP |
| `docs/nextjs-api-routes.md` | Plan for httpOnly cookies migration |
| `docs/cloudflare-tunnels.md` | Webhook testing setup |

## Security Features

- JWT with access (15m) + refresh (7d) tokens
- Rate limiting: 3 req/s, 20 req/10s, 100 req/min
- Helmet security headers
- bcrypt password hashing
- Zod validation on all inputs
- CORS restricted to frontend origin
- Webhook signature validation (MercadoPago)
- JWT exception filter with semantic error codes

---
> Source: [MarcosRehtanz/ecommerce-backend](https://github.com/MarcosRehtanz/ecommerce-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
