## leiaguard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LeiaGuard is a WireGuard VPN Management System built with Next.js 15. It allows users to register with invitation codes, generate VPN credentials for multiple devices, and provides administrators with user management capabilities.

## Commands

### Development
- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm run start` - Start production server

### Code Quality
- `npm run lint` - Run ESLint
- `npm run lint:fix` - Run ESLint with auto-fix
- `npm run types` - Run TypeScript type checking
- `npm run format:write` - Format code with Prettier
- `npm run clean` - Run both lint:fix and format:write

### Database
- `npm run db:generate` - Generate Drizzle migration files
- `npm run db:migrate` - Run migrations
- `npm run db:seed` - Seed database with test data

### Testing
- `npm run test` - Run all tests (unit + e2e)
- `npm run test:unit` - Run Jest unit tests
- `npm run test:e2e` - Run Playwright e2e tests

### Deployment
- `./deploy-hetzner.sh` - Deploy to Hetzner VPS (preserves database)

## Architecture

This is a Next.js 15 application using the App Router with SQLite database and NextAuth.js for authentication.

### Route Structure
- `/app/(unauthenticated)` - Public routes
  - `(marketing)` - Landing pages, pricing, features, about, contact
  - `/login` - User login
  - `/register` - User registration with invitation codes
- `/app/(authenticated)` - Protected routes requiring authentication
  - `dashboard` - Main user dashboard
  - `dashboard/devices` - Device management (add, view, delete)
  - `dashboard/admin` - Admin panel for user and invitation code management
- `/app/api` - API routes
  - `/api/auth` - NextAuth.js authentication routes
  - `/api/devices` - Device CRUD and config/QR generation
  - `/api/admin` - Admin endpoints for users and invitation codes

### Key Patterns
- **API Routes** in `/app/api` for device management, auth, and admin operations
- **Database Schema** in `/db/schema` using Drizzle ORM with SQLite (better-sqlite3)
- **UI Components** in `/components/ui` from Shadcn UI library
- **Authentication** handled by NextAuth.js with credentials provider
- **WireGuard Integration** - Automatic peer management on server

### Data Flow
1. Authentication state managed by NextAuth.js with session-based auth
2. User and device data stored in SQLite via Drizzle ORM
3. Invitation codes control user registration
4. WireGuard configs generated with QR codes for mobile setup

### Key Files
- `/db/schema/` - Database schema definitions (users, devices, invitation_codes)
- `/lib/auth.ts` - NextAuth.js configuration
- `/app/api/devices/` - Device management API routes
- `/app/api/admin/` - Admin API routes

### Environment Variables Required
- `NEXTAUTH_SECRET` - Secret for NextAuth.js session encryption
- `NEXTAUTH_URL` - Application URL (e.g., http://localhost:3000)
- `AUTH_TRUST_HOST` - Set to true for trusted hosts
- `WIREGUARD_SERVER_IP` - WireGuard server IP address
- `WIREGUARD_SERVER_DOMAIN` - WireGuard server domain
- `WIREGUARD_SERVER_PUBLIC_KEY` - Server's WireGuard public key
- `WIREGUARD_SERVER_PORT` - WireGuard port (default: 51820)
- `WIREGUARD_NETWORK_RANGE` - VPN network range (e.g., 10.0.0.0/24)

---
> Source: [arashvakil/LeiaGuard](https://github.com/arashvakil/LeiaGuard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
