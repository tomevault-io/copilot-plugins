## nextjs-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `pnpm dev` - Start development server with Turbopack
- `pnpm build` - Build the application for production
- `pnpm start` - Start the production server

## Architecture

This is a Next.js App Router dashboard application following the structure:

- **App Router Structure**: Uses Next.js 13+ app directory structure with TypeScript
- **UI Components**: Components organized in `app/ui/` with feature-based folders (dashboard, invoices, customers)
- **Data Layer**: Database operations and type definitions in `app/lib/`
  - `definitions.ts` - TypeScript type definitions for data models (User, Customer, Invoice, Revenue)
  - `data.ts` - Database query functions
  - `utils.ts` - Utility functions
- **API Routes**: Server actions in `app/query/route.ts` and database seeding in `app/seed/route.ts`

## Technology Stack

- **Framework**: Next.js with App Router
- **Styling**: Tailwind CSS with custom blue color palette and shimmer animations
- **Authentication**: NextAuth 5.0 beta
- **Database**: PostgreSQL with `postgres` driver
- **Validation**: Zod for schema validation
- **Icons**: Heroicons React
- **Package Manager**: pnpm (uses `onlyBuiltDependencies` for bcrypt and sharp)

## Key Patterns

- TypeScript types are manually defined rather than generated from ORM
- Uses `clsx` for conditional CSS classes
- Custom Tailwind configuration includes 13-column grid and shimmer keyframes
- UI components follow atomic design with reusable elements in feature folders

---
> Source: [mojakaz/nextjs-dashboard](https://github.com/mojakaz/nextjs-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
