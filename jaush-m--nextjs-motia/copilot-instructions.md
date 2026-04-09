## nextjs-motia

> This document provides a comprehensive architecture and design for the `resorts` project—a production-ready resort booking platform built with a type-safe, modular, and scalable full-stack architecture.

# Gemini Project: Resorts

This document provides a comprehensive architecture and design for the `resorts` project—a production-ready resort booking platform built with a type-safe, modular, and scalable full-stack architecture.

---

## Project Overview

The `resorts` project is a **monorepo** managed with `pnpm` and `Turborepo`, containing:

- **`apps/web`** – Next.js frontend application
- **`apps/api`** – NestJS backend service
- **`packages/db`** – Shared Drizzle ORM models and types
- **`packages/ui`** – Shared React components (shadcn/ui)
- **`packages/configs`** – Shared ESLint, Tailwind, and TypeScript configs

**Monorepo Goals:**

- Shared **Drizzle ORM** types between backend and frontend for end-to-end type safety.
- Reusable **UI components** across web and admin interfaces.
- Modular services and packages with strict separation of concerns.
- Turborepo caching and pipeline optimization for fast builds.

---

## Tech Stack

**Frontend**

- Next.js (App Router + Server Components)
- TypeScript
- shadcn/ui (shared UI components)
- Tailwind CSS
- SEO-first architecture
- Client components only for interactivity (e.g., booking sidebar, date pickers)
- Realtime via Motia Streams

**Backend**

- Motia (modular architecture)
- PostgreSQL
- Drizzle ORM (type-safe schema, queries, and shared types)
- BetterAuth (JWT/session authentication)
- tRPC for connection between frontend and backend
- Clean architecture (Controllers → Services → Repositories)

**CMS / Admin**

- Admin CMS for resorts, rooms, rates, availability, blog/media
- Admin integrates directly with shared DB or local API

**Realtime & Caching**

- Realtime events for:
  - Booking locks (prevent double booking)
  - Availability updates
  - Admin live updates

- Redis:
  - horizontal scaling
  - Caching availability and read-heavy data

---

## Monorepo Structure

```
resorts/
├─ apps/
│  ├─ web/            # Next.js frontend
│  └─ api/            # NestJS backend
├─ packages/
│  ├─ db/             # Shared Drizzle ORM models & types
│  ├─ ui/             # Shared UI components
│  ├─ configs/        # ESLint, TS, Tailwind
├─ turbo.json
├─ pnpm-workspace.yaml
├─ gemini.md
```

---

## Shared Drizzle ORM (`packages/db`)

- Centralized schema for **type safety** in frontend and backend.
- Contains:
  - `resorts`, `rooms`, `rates`, `availability`, `users`, `bookings`, `payments`

- Exported types can be imported in **Next.js** and **NestJS**:

```ts
import { Resort, Room, Booking } from "@resorts/db";
```

- Advantages:
  - Frontend can pre-validate forms using shared types.
  - Backend enforces consistency via shared models.
  - Reduces duplication and errors in type definitions.

---

## Authentication (BetterAuth)

- Uses **JWT-based authentication** for frontend session management.
- Role-Based Access Control (RBAC):
  - **Guest** – limited search & booking capabilities
  - **User** – booking management, favorites, profile
  - **Admin** – CRUD on resorts, rooms, rates, bookings, blog

- **Security Measures:**
  - No mass assignment
  - DTO validation with class-validator
  - CSRF/XSS protection
  - Rate limiting
  - Session expiration

---

## Next.js Architecture

**App Router Structure:**

```
app/
├─ layout.tsx
├─ page.tsx
├─ resorts/
│  ├─ [id]/
│  │  ├─ page.tsx
│  │  └─ layout.tsx
├─ bookings/
│  ├─ checkout/
│  └─ confirmation/
├─ auth/
│  ├─ login/
│  └─ register/
├─ dashboard/
│  └─ my-bookings/
```

**Component Strategy:**

- Server Components by default for SEO and performance.
- Client Components for:
  - Booking sidebar
  - Interactive date pickers
  - Payment forms

- Data fetching:
  - `serverActions` for mutations
  - `fetch` with DRF / NestJS REST endpoints

**SEO Strategy:**

- Dynamic metadata per resort page
- JSON-LD schema for resorts and rooms
- OpenGraph / Twitter cards
- Sitemap generation
- Robots.txt configured for public pages

---

## NestJS Architecture

**Modules:**

```
modules/
├─ auth/
├─ users/
├─ resorts/
├─ rooms/
├─ rates/
├─ availability/
├─ bookings/
├─ payments/
├─ notifications/
```

**Service Boundaries:**

- Controllers handle routing/validation
- Services handle business logic
- Repositories handle Drizzle queries
- DTOs and class-validator for input validation
- Booking transactions:
  - Atomic locking using Socket.io + DB transaction
  - Idempotency for payment handling

---

## Database Design (Drizzle ORM)

**Entities:**

- Resorts → Rooms → Rates → Availability (per date)
- Users → Bookings → Payments

**Relationships:**

- One-to-many: Resorts → Rooms
- One-to-many: Rooms → Rates
- Many-to-one: Bookings → Users
- One-to-one: Bookings → Payments
- Availability: Date-based, includes inventory count

**Indexes:**

- Unique constraint on room/date
- Index on bookings for user queries
- Full-text index for resort search

**Booking Lock Strategy:**

- Temporary booking lock in
- Expire if payment not completed in X minutes
- Prevents double-booking

**Soft Deletes:**

- Applied to Users, Bookings, Rooms for auditability

---

## Realtime Design (Socket.io)

**Use Cases:**

- Booking locks and availability
- Admin updates (live booking/payment changes)
- Notifications

**Channels:**

- `availability:<roomId>` – broadcast availability changes
- `booking:<bookingId>` – broadcast booking status
- `admin:<adminId>` – internal admin updates

**Scaling Considerations:**

- Redis adapter for multiple nodes
- Rate-limited broadcast channels

---

## UX/UI Design Principles

- Clean, modern, minimal interface (Airbnb/Booking.com inspired)
- Mobile-first responsive design
- Sticky booking sidebar
- Date pickers with availability indicators
- Clear pricing breakdown (nightly rates, taxes, fees)
- Skeleton loaders for async content
- Trust signals: reviews, policies, badges
- Error handling and empty states

---

## Critical Booking Flow

1. User selects resort and date range
2. System checks availability
3. User selects room & rate → shows price breakdown
4. Booking lock applied via Socket.io
5. Guest or Registered User enters payment info
6. Payment processed (idempotent)
7. Booking confirmation displayed & email sent
8. Handle:
   - Loading states
   - Payment failures
   - Expired booking locks

---

## Performance & Scalability

- Redis caching for availability & public pages
- Edge caching via Next.js
- Database indexing for searches & availability
- Background jobs:
  - Email confirmations
  - Cleanup expired locks

- Image optimization
- Modular services for horizontal scaling

---

## Deployment Architecture

**VPS-friendly Docker Compose setup:**

```yaml
services:
  web:
    build: ./apps/web
    ports:
      - "3000:3000"
  backend:
    build: ./apps/backend
    ports:
      - "8080:8080"
  postgres:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
  redis:
    image: redis:7
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
volumes:
  pgdata:
```

- Nginx as reverse proxy for web and API
- TLS termination via Certbot / Let's Encrypt
- Environment variables managed per service
- Logging and monitoring using Docker logs / Prometheus

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jaush-M)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jaush-M)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
