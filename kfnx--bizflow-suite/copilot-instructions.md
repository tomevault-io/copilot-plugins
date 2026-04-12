## bizflow-suite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MySTI is a business document generation application built for PT San Traktor Indonesia. It's a Next.js application with MySQL database using Drizzle ORM, implementing a complete business document workflow for quotations, invoices, delivery notes, and inventory management.

## Development Commands

### Package Manager

Use `pnpm` for all package management operations.

### Core Development

- `pnpm dev` - Start development server on http://localhost:3000
- `pnpm build` - Build for production
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint
- `pnpm lint:fix` - Run ESLint with automatic fixes
- `pnpm typecheck` - Run TypeScript type checking (corrected command)
- `pnpm format` - Format code with Prettier

### Database Operations

- `pnpm db:generate` - Generate new Drizzle migrations
- `pnpm db:migrate` - Run pending migrations
- `pnpm db:push` - Push schema changes directly to database
- `pnpm db:studio` - Open Drizzle Studio (database GUI)
- `pnpm db:seed` - Seed database with sample data
- `pnpm db:truncate` - Truncate all database tables
- `pnpm db:drop` - Drop database tables (use with caution)

### Docker & MinIO

- `pnpm docker:dev` - Start MySQL and MinIO containers for development
- `pnpm init:minio` - Initialize MinIO buckets and configuration
- `pnpm test:minio` - Test MinIO connection and setup

## Architecture Overview

### Application Structure

- **App Router**: Uses Next.js 14 App Router with TypeScript
- **Layout Groups**:
  - `(auth)` - Authentication pages (login, register, verification)
  - `(main)` - Main application with sidebar layout
- **Database**: MySQL with Drizzle ORM for type-safe database operations
- **UI Framework**: AlignUI Design System with Radix UI components and Tailwind CSS
- **State Management**: Jotai for global state, TanStack Query for server state
- **File Storage**: MinIO for document and file storage
- **Monitoring**: Sentry for error tracking and performance monitoring

### Database Schema

Complete business document workflow with proper foreign key relationships:

**Core Entities:**

- `users` - User accounts with roles and permissions system
- `customers` + `customer_contact_persons` - Customer management
- `suppliers` + `supplier_contact_persons` - Supplier management
- `warehouses` - Warehouse locations with stock tracking
- `products` - Product catalog with supplier relationships and stock management
- `branches` - Company branch locations

**Master Data:**

- `brands` - Product brands
- `machine_types` - Equipment categories
- `machine_model` - Product model specifications
- `part_numbers` - Parts catalog
- `unit_of_measures` - Units for inventory (pcs, kg, etc.)

**Document Flow:**

- `quotations` + `quotation_items` - Customer quotations with approval workflow
- `invoices` + `invoice_items` - Invoices (can be generated from quotations)
- `delivery_notes` + `delivery_note_items` - Delivery documentation
- `imports` + `import_items` - Goods received from suppliers
- `transfers` + `transfer_items` - Warehouse-to-warehouse transfers

### Key Components

- **Sidebar Navigation**: `/components/sidebar.tsx` - Main navigation
- **Header Components**: `/components/header.tsx` and `/components/header-mobile.tsx`
- **UI Components**: `/components/ui/` - Reusable UI components based on AlignUI
- **Search System**: `/components/search.tsx` - Global command menu
- **Data Tables**: Uses TanStack Table for data display with filtering and pagination
- **PDF Generation**: Uses `@react-pdf/renderer` for document generation

## Environment Configuration

Required environment variables:

**Database:**

- `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` - Database configuration
- `DB_SSL` - SSL configuration ("required", "preferred", or undefined)

**Authentication:**

- `NEXTAUTH_URL`, `NEXTAUTH_SECRET` - NextAuth configuration

**File Storage:**

- MinIO configuration for document storage

**Monitoring:**

- Sentry configuration for error tracking

## Key Development Patterns

### Code Style Guidelines (from .cursor/rules)

- Use `pnpm` for package manager and task runner
- Use `@remixicon/react` for importing icons
- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`)
- Structure files: exported components, subcomponents, helpers, static content, types
- Use lowercase with dashes for directory names (e.g., `components/auth-wizard`)
- Minimize use of `'use client'`, `useEffect`, and `setState`; favor React Server Components
- Implement guard clauses for error handling and early returns
- Use responsive design with mobile-first approach

### Database Queries

- Use Drizzle ORM with TypeScript for all database operations
- Schema defined in `/lib/db/schema.ts` with proper relations
- Database connection in `/lib/db/index.ts`
- Migrations stored in `/lib/db/migrations/`

### Styling

- Tailwind CSS with custom AlignUI design system
- Custom color palette and typography defined in `tailwind.config.ts`
- Theme support with next-themes

### API Routes

- RESTful API structure in `/app/api/`
- Zod schemas for validation in `/lib/validations/`
- Proper HTTP status codes and error handling
- OpenAPI documentation generated automatically

### State Management

- Use Jotai for global state management
- TanStack Query for server state and caching
- Custom hooks in `/hooks/` for data fetching patterns

## Business Logic

The application follows a standard business document workflow:

1. **Product Management** - Products with supplier information, categorized as serialized, non-serialized, or bulk
2. **Inventory Control** - Stock managed via imports from suppliers and warehouse transfers
3. **Quotation Process** - Create quotations for customers with approval workflow
4. **Invoice Generation** - Convert approved quotations to invoices or create standalone invoices
5. **Delivery Management** - Create delivery notes for shipments
6. **Document Numbering** - Automatic sequential numbering for all document types

## Testing and Quality

Always run these commands before committing:

- `pnpm lint` - Check for linting errors
- `pnpm typecheck` - Run TypeScript type checking
- `pnpm build` - Ensure the application builds successfully
- Test database operations with `pnpm db:studio` to verify schema changes

Use `pnpm lint:fix` and `pnpm format` to automatically fix code style issues.

## Docker Environment

The application is fully dockerized with:

- Production build optimization (`output: 'standalone'`)
- MySQL database with initialization scripts
- MinIO for file storage
- Environment variable configuration for container deployment
- Development and production profiles in `docker-compose.yml`

## File Organization

- `/app/` - Next.js App Router pages and API routes
- `/components/` - Reusable React components
- `/lib/` - Utility functions, database schema, and configurations
- `/hooks/` - Custom React hooks for data fetching
- `/types/` - TypeScript type definitions
- `/docs/` - Project documentation
- `/public/` - Static assets (images, icons, flags)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kfnx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kfnx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
