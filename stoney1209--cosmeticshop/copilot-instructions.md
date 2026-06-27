## cosmeticshop

> - `npm run dev` - Start development server (http://localhost:3000)

# Development Guide

## Commands

- `npm run dev` - Start development server (http://localhost:3000)
- `npm run build` - Generate Prisma client, then build (required before deployment)
- `npm run lint` - Run ESLint
- `npm run postinstall` - Auto-runs `prisma generate` (also runs after npm install)

## Architecture

- **Framework**: Next.js 16.2.4 (App Router, beta)
- **Database**: Prisma with PostgreSQL (DATABASE_URL required in .env)
- **Styling**: Tailwind CSS v4 with shadcn/ui components
- **State**: Zustand for cart, React Context for customer session

## Route Structure

- `(shop)/` - Customer-facing pages (tienda, producto/[slug], cuenta/*, etc.)
- `(admin)/` - Admin dashboard (pedidos, productos, usuarios, facturacion, etc.)
- `src/app/actions/` - Server Actions (products, orders, customers, inventory, etc.)
- `src/app/api/` - API routes (auth, v1/*)

## Auth System

- **Admin**: NextAuth with credentials provider (User model with ADMIN/OPERATOR roles)
- **Customers**: Custom session via `src/lib/customer-session.ts` (Customer model with email/password)

## Database

- `npx prisma studio` - Open Prisma database GUI
- `npx prisma db push` - Push schema to database
- `npx prisma generate` - Generate client (auto-runs on postinstall and build)

## Key Files

- `prisma/schema.prisma` - Full database schema (User, Customer, Product, Order, Invoice, etc.)
- `src/store/cart.ts` - Zustand cart store
- `src/lib/auth.ts` - NextAuth configuration
- `src/lib/customer-session.ts` - Customer auth utilities
- `src/middleware.ts` - Route protection

---
> Source: [Stoney1209/CosmeticShop](https://github.com/Stoney1209/CosmeticShop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
