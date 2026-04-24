## lesgo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LesGo is an Indonesian private tutoring marketplace platform that connects parents/students with private tutors based on location and subject. The name "LesGo" combines "Les" (Indonesian for "tutoring") and "Let's Go".

## Tech Stack

- **Framework**: Next.js 16.1.6 with App Router
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS v4
- **UI Components**: shadcn/ui (New York style)
- **Database**: PostgreSQL + PostGIS (Supabase)
- **Authentication**: Supabase Auth (OTP via WhatsApp)
- **Payment**: Midtrans (Snap)
- **Maps**: Google Maps API
- **State Management**: Zustand
- **Forms**: React Hook Form + Zod

## Available Commands

```bash
# Development
npm run dev          # Start development server on localhost:3000

# Build & Production
npm run build        # Build for production
npm run start        # Start production server

# Code Quality
npm run lint         # Run ESLint (configured with next/core-web-vitals)
```

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes
│   │   ├── payments/      # Midtrans payment creation & webhook
│   │   └── tutors/search  # Geospatial tutor search
│   ├── parent/            # Parent-facing routes
│   │   ├── search/        # Tutor search page
│   │   ├── tutor/[id]/    # Tutor profile & booking
│   │   └── payment/[id]/  # Payment page
│   ├── tutor/             # Tutor-facing routes
│   │   └── dashboard/     # Tutor dashboard
│   ├── login/             # Login page
│   ├── register/          # Registration page
│   └── layout.tsx         # Root layout with PWA meta tags
├── components/
│   ├── auth/              # LoginForm, RegisterForm, RoleSelector
│   ├── booking/           # BookingForm
│   ├── search/            # SubjectSelector, LocationPicker, TutorCard
│   ├── tutor/             # BookingRequests, EarningsCard
│   └── ui/                # shadcn/ui components
├── lib/
│   ├── supabase.ts        # Client-side Supabase client
│   ├── supabase-server.ts # Server-side Supabase client (service role)
│   ├── midtrans.ts        # Midtrans payment client
│   └── utils.ts           # cn() helper for Tailwind
└── types/
    ├── index.ts           # TypeScript interfaces
    └── midtrans.d.ts      # Midtrans type declarations

supabase/migrations/       # Database migrations
├── 001_initial_schema.sql
├── 002_rls_policies.sql
└── 003_search_function.sql
```

## Architecture Patterns

### Role-Based Routing
The platform has two user roles: `parent` and `tutor`. Routes are organized under `/parent/*` and `/tutor/*` respectively. Role selection happens during registration.

### Supabase Client Pattern
- **Client-side** (`src/lib/supabase.ts`): Uses anon key for browser operations, respects RLS policies
- **Server-side** (`src/lib/supabase-server.ts`): Uses service role key for API routes, bypasses RLS

### Geospatial Search
Uses PostGIS extension with `ST_DWithin` for radius filtering and `ST_Distance` for distance calculation. The `search_tutors` SQL function handles the geospatial query.

### Payment Flow (Escrow Model)
- 15% platform commission on each booking
- Parent pays full amount to LesGo via Midtrans
- Tutor receives earnings minus commission to their wallet
- Webhook handler at `/api/payments/webhook` updates payment status

### Database Schema (7 core tables)
1. `profiles` - User profiles (extends Supabase auth.users)
2. `tutor_profiles` - Tutor-specific data (subjects, rate, location, verification)
3. `bookings` - Booking sessions with status tracking
4. `payments` - Payment records linked to Midtrans
5. `reviews` - Tutor ratings and comments
6. `availability` - Tutor schedule availability
7. `tutor_wallets` + `wallet_transactions` - Earnings tracking

All tables have Row Level Security (RLS) enabled.

## Environment Variables

Required in `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
MIDTRANS_SERVER_KEY=
MIDTRANS_CLIENT_KEY=
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
```

## Key Implementation Details

### Authentication
- Uses Supabase Auth with phone OTP (WhatsApp)
- No traditional password-based auth
- Middleware protects `/parent/*` and `/tutor/*` routes

### Booking Status Flow
```
pending_payment → paid → confirmed → in_progress → completed
                          ↓
                    cancelled / declined
```

### TypeScript Path Aliases
- `@/*` maps to `src/*`
- Used consistently across the codebase for imports

### shadcn/ui Configuration
- Style: "new-york"
- Base color: slate
- CSS variables enabled
- Icon library: lucide

## Documentation

- `docs/2026-02-16-lesgo-design.md` - Design specification with business model, user flows, error handling
- `docs/2026-02-16-lesgo-implementation.md` - Detailed implementation plan with 12 tasks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nabePi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
