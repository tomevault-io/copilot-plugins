## yimei

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a medical beauty clinic website template built with Next.js 14, React 18, TypeScript, and Prisma. It features a public-facing website for services and booking, plus a comprehensive admin dashboard for managing appointments, customers, orders, and reports.

## Development Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint (currently ignores errors during builds)

## Database Commands

- `npm run db:seed` - Seed database with initial data
- `npm run db:reset` - Reset database and re-seed
- `npm run db:studio` - Open Prisma Studio for database management

## Architecture

### Tech Stack
- **Framework**: Next.js 14 with App Router
- **UI**: Radix UI components + shadcn/ui + Tailwind CSS
- **Database**: SQLite with Prisma ORM
- **Authentication**: JWT + bcryptjs for admin auth
- **Forms**: React Hook Form + Zod validation
- **Icons**: Lucide React

### Directory Structure

- `app/` - Next.js App Router pages
  - `admin/` - Admin dashboard pages (bookings, customers, orders, reports, settings, logs)
  - `booking/` - Public booking system
  - `services/`, `cases/`, `news/`, `contact/`, `about/` - Public pages
- `components/` - Reusable React components
  - `ui/` - shadcn/ui components
  - `admin/` - Admin-specific components
  - `layout/` - Header/Footer components
- `lib/` - Utilities and configurations
  - `admin/` - Admin utilities (auth, middleware)
  - `generated/prisma/` - Generated Prisma client
- `prisma/` - Database schema and migrations

### Database Schema

Key models:
- `Admin` - Admin users with roles (SUPER_ADMIN, ADMIN, OPERATOR)
- `Booking` - Customer appointments with detailed info
- `Order` - Payment tracking linked to bookings
- `AdminLog` - Audit trail for admin actions
- `SystemSetting` - Configuration management

### Path Aliases
- `@/*` - Root directory alias for imports

### UI Configuration
- Uses shadcn/ui with "new-york" style
- Tailwind with CSS variables for theming
- Lucide React for icons
- Geist fonts (Sans + Mono)

### Admin Features
- Dashboard with analytics and charts (using Recharts)
- Booking management with status tracking
- Customer management with detailed profiles
- Order/payment tracking
- System logs and audit trails
- Settings management for admins and system config

## Important Notes

- TypeScript errors are ignored during builds (`ignoreBuildErrors: true`)
- ESLint errors are ignored during builds (`ignoreDuringBuilds: true`)
- Images are unoptimized (`unoptimized: true`)
- Database client is generated to `lib/generated/prisma/`
- Uses SQLite for development (check `.env` for DATABASE_URL)

## 请始终使用简体中文回答

---
> Source: [map-c/yimei](https://github.com/map-c/yimei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
