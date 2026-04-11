## booking-lpbc2026

> **Phantom Equinox** is a premium Meeting Room Booking Application built with modern web technologies. It is designed to handle multi-tenant corporate environments (Companies), allowing users to book meeting rooms across different buildings and floors.

# Phantom Equinox - Project Context

## Overview
**Phantom Equinox** is a premium Meeting Room Booking Application built with modern web technologies. It is designed to handle multi-tenant corporate environments (Companies), allowing users to book meeting rooms across different buildings and floors.

## Technology Stack

### Core
- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript
- **Style:** Tailwind CSS 4
- **Animation:** Framer Motion

### Backend & Database
- **Platform:** Supabase (PostgreSQL)
- **Auth:** Supabase Auth (with Middleware protection)
- **Email:** Resend (implied by dependencies)

### UI Components
- **Icons:** Lucide React
- **Forms:** React Hook Form + Zod resolvers
- **Notifications:** Sonner
- **Theming:** next-themes (System, Light, Dark)

## Architecture & Directory Structure

```
/
├── app/                  # Next.js App Router pages and layouts
│   ├── admin/            # Administrative interfaces
│   ├── bookings/         # Booking management
│   ├── login/            # Authentication pages
│   ├── rooms/            # Room browsing and details
│   ├── schedule/         # Calender/Schedule views
│   ├── settings/         # User settings
│   ├── api/              # API Routes (if any)
│   ├── globals.css       # Global styles and Tailwind directives
│   └── layout.tsx        # Root layout with Providers
├── components/           # Reusable UI components
├── db/                   # Database schemas and SQL scripts
│   └── schema.sql        # Main database definition
├── lib/                  # Utility functions and shared logic
│   └── supabase/         # Supabase client initialization
├── public/               # Static assets
└── types/                # TypeScript type definitions
```

## Data Model

### Core Entities
1.  **Companies**: Tenants of the system.
2.  **Profiles**: Users linked to `auth.users`, containing role (`admin` vs `user`) and company association.
3.  **Buildings**: Physical locations owned by companies.
4.  **Floors**: Levels within buildings.
5.  **Rooms**: Bookable entities with capacity and amenities.
6.  **Bookings**: Reservations linking Users and Rooms with time slots and status.

### Authentication & Authorization
- **RLS (Row Level Security)** is enabled on all tables.
- **Policies:**
    - Public read access for Buildings, Floors, and Rooms.
    - Users can see all bookings (to avoid double booking) but only edit their own.
    - Admins can manage all bookings.

## Development

### Commands
- \`npm run dev\`: Start development server
- \`npm run build\`: Build for production
- \`npm run lint\`: Run ESLint

### Key Conventions
- **Styling**: Utility-first with Tailwind.
- **State Management**: Server Components for data fetching where possible; Client Components for interactivity.
- **Database Access**: Direct Supabase calls or Server Actions (implied by typical Next.js 14+ patterns, though `actions` folder exists).

## Current Status
- **Auth**: Implemented via Supabase Middleware.
- **Database**: Schemas defined for core entities.
- **Routing**: Skeleton routes exist for Admin, Bookings, Login, Rooms, Schedule, Settings.

## Admin Guide

### 1. Setup
To enable the Admin backend, you must run the following SQL scripts in your Supabase SQL Editor:
1.  `db/01_auth_triggers.sql`: **Required**. Creates a user profile automatically upon sign-up.
2.  `db/02_admin_policies.sql`: **Required**. Secures the database so only Admins can manage resources.

### 2. Creating an Admin
There is no UI to create the *first* admin. You must promote a user manually:
1.  Sign up in the application.
2.  Open `db/03_promote_admin.sql`.
3.  Replace the email with your email and run the script in Supabase.

### 3. Features
Once promoted, you can access `/admin` to:
- **Dashboard**: Approve or Reject pending bookings.
- **Buildings**: Manage office buildings (`/admin/buildings`).
- **Floors**: Manage floors within buildings (`/admin/floors`).
- **Rooms**: Manage meeting rooms (`/admin/rooms`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DefinitionGroup)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DefinitionGroup)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
