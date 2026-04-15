## app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Talk-To-My-Lawyer is a Next.js 15 application for generating professional legal letters with AI. The app features role-based access control (admin, employee, subscriber), subscription management, and AI-powered letter generation with PDF export capabilities.

## Development Commands

```bash
# Development
npm run dev              # Start development server (localhost:3000)
npm run build           # Build for production
npm run start           # Start production server
npm run lint            # Run ESLint
npm run typecheck       # Run TypeScript type checking

# Database
npm run db:push         # Push schema changes to Supabase
npm run db:gen-types    # Generate TypeScript types from database schema
```

## Architecture Overview

### Tech Stack
- **Framework**: Next.js 15 with App Router
- **Database**: Supabase (PostgreSQL with real-time features)
- **Styling**: Tailwind CSS with custom animations
- **UI Components**: Radix UI primitives with custom variants
- **AI**: OpenAI API for letter generation
- **PDF Generation**: pdf-lib for creating legal documents
- **Authentication**: Supabase Auth with role-based access

### Key Directories
- `app/` - Next.js app router pages and API routes
- `lib/` - Shared utilities (auth, Supabase clients, helpers)
- `components/` - Reusable UI components
- `supabase/` - Database configuration and migrations

### Database Schema
The application uses a multi-role system with these main tables:
- `profiles` - User profile information
- `user_roles` - Role assignments (user/employee/admin)
- `subscriptions` - User subscription management
- `letters` - Generated legal letters
- `commissions` - Employee referral tracking

### Authentication & Authorization
- Uses middleware for route protection (`middleware.ts`)
- Role-based access control implemented in `lib/auth.ts`
- Three user roles:
  - `user` (subscriber) - Can generate letters within subscription limits
  - `employee` - Can manage subscriber letters and earn commissions
  - `admin` - Full system access

### Supabase Integration
- Client-side: Singleton pattern in `lib/supabase/client.ts`
- Server-side: Server client in `lib/supabase/server.ts`
- Type-safe queries with auto-generated types in `lib/types/database.ts`

### Key Features
- AI-powered legal letter generation using OpenAI
- PDF generation and email delivery
- Subscription-based quota system
- Employee commission tracking
- Role-based dashboard interfaces
- Real-time updates with Supabase subscriptions

## Environment Setup

Required environment variables:
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
OPENAI_API_KEY=your_openai_api_key
```

## Deployment

The app is configured for deployment with:
- Netlify (primary) - with `@netlify/plugin-nextjs`
- Vercel support
- Supabase for database and auth hosting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aqeelwebbing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
