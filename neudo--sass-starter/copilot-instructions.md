## sass-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hector Analytics - A privacy-first web analytics SaaS platform built with Next.js and Supabase, providing an alternative to Google Analytics without using cookies.

## Essential Commands

```bash
# Development
pnpm dev          # Start development server with Turbopack on http://localhost:3000

# Build & Production
pnpm build        # Create production build
pnpm start        # Start production server

# Code Quality
pnpm lint         # Run ESLint checks
```

## Architecture Overview

### Tech Stack

- **Framework**: Next.js with App Router
- **Database**: Supabase (PostgreSQL)
- **UI**: shadcn/ui components with Radix UI
- **Styling**: Tailwind CSS v4 with CSS variables
- **Auth**: Supabase Auth with cookie-based sessions
- **Email**: Resend API
- **Forms**: React Hook Form + Zod validation

### Project Structure

```
app/                    # Next.js App Router pages
├── api/track/         # Analytics tracking endpoint
├── auth/              # Authentication flows
├── dashboard/[domain]/ # Domain-specific analytics dashboard
└── (marketing pages)   # Public pages

components/
├── ui/                # shadcn/ui reusable components
├── SiteData/          # Analytics visualization components
└── auth-*.tsx         # Authentication forms

lib/
├── supabase/          # Supabase client configurations
│   ├── client.ts      # Browser client
│   ├── server.ts      # Server client (RSC)
│   └── admin.ts       # Service role client
└── geo.ts             # IP geolocation utilities
```

### Key Patterns

#### Supabase Client Usage

- Use `createClient()` from `lib/supabase/server.ts` in Server Components
- Use `createClient()` from `lib/supabase/client.ts` in Client Components
- Use `createAdminClient()` from `lib/supabase/admin.ts` for admin operations

#### Authentication Flow

- All dashboard routes require authentication via middleware
- Session handled through cookies (no JWT in localStorage)
- Auth forms use Server Actions for form submission

#### Analytics Tracking

- Privacy-first: No cookies used
- Tracking script: `/public/script.js`
- API endpoint: `/api/track` handles session upserts
- Geolocation via MaxMind GeoIP2 database

### Database Schema (Key Tables)

- `sites` - Website configurations
- `sessions` - Analytics data with geolocation
- `waitlist` - Email signups

### Environment Variables

Required in `.env.local`:

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_OR_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
RESEND_API_KEY=
```

### Component Guidelines

- UI components use shadcn/ui patterns (found in `components/ui/`)
- Forms use React Hook Form with Zod schemas
- All components are TypeScript with proper typing
- Use `cn()` utility for className merging

### Styling

- Tailwind CSS v4 with CSS-first configuration
- Theme variables defined in `app/globals.css`
- Dark/light mode via next-themes
- Primary brand color: #3d9dbd (blue)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Neudo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
