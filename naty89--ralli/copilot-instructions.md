## ralli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Ralli** is an event transportation management platform that provides a closed, event-based queue system for coordinating safe rides during parties and events (primarily for fraternities). It features three distinct user roles: Admin (event management), Driver (ride fulfillment), and Rider (ride requests).

**Tech Stack**: Next.js 14 (App Router), React, TypeScript, TailwindCSS, Supabase (PostgreSQL + Auth + Realtime), Google Maps API

## Common Development Commands

```bash
# Start development server (runs on http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint

# Run integration tests (idempotent rides API)
npm run test:integration -- https://myapp.vercel.app <event_id>
```

**Environment Setup**: Copy `.env.example` to `.env.local` and fill in Supabase URL/keys and Google Maps API key.

## High-Level Architecture

### Frontend Structure

- **`app/admin/`**: Admin dashboard (event creation, driver management, ride assignment queue)
- **`app/driver/`**: Driver interface (status toggle, route navigation, ride completion flow)
- **`app/rider/`**: Rider request form (pickup location selection, ride tracking)
- **`components/`**: Reusable UI components and specialized components (PlacesAutocomplete, DriverLocationMap, etc.)
- **`lib/`**: Core logic layer with service functions

### Backend Architecture

**API Routes** (`app/api/`):
- `POST/GET /api/rides`: Core idempotent ride creation with rate limiting and existing ride detection
- `POST /api/rider/confirm-presence`: Rider consent confirmation before ride creation
- `POST /api/cron/process-noshow`: Automatic no-show processing (serverless cron trigger)
- `POST /api/seed`: Database seeding endpoint

**Service Layer** (`lib/services/`):
- **rideGuardService.ts**: Rider identification (phone/client_id based hashing), rate limiting, idempotency guards
- **rides.ts**: Ride request operations
- **dispatchService.ts**: Smart dispatch algorithm (haversine distance, optimal driver matching, ride state machine)
- **drivers.ts**: Driver management and availability
- **events.ts**: Event CRUD and access control
- **auth.ts**: Authentication and profile management
- **etaService.ts**: ETA calculations
- **analyticsService.ts**: Event metrics and reporting
- **safetyService.ts**: Safety features (emergency contacts, consent verification)
- **batchService.ts**: Batch operations for rides
- **emergencyService.ts**: Emergency handling

### Database Design

**Key Tables**:
- `profiles`: User profiles linked to auth.users (role: admin/driver)
- `events`: Event metadata with unique access codes
- `drivers`: Driver assignments per event (online status, location)
- `ride_requests`: Ride requests with state machine (waiting → assigned → arrived → in_progress → completed/cancelled)

**Key Columns to Know**:
- `rider_identifier_hash`: Stable SHA256 hash of `event_id:phone` or `event_id:client_id` for idempotency
- `rider_phone_normalized`: 10+ digit normalized phone for identification priority
- `status`: Controlled enum: `waiting`, `assigned`, `arrived`, `in_progress`, `completed`, `cancelled`

**RLS Policies**: Row-level security is enabled on all tables. Admins manage only their own events; drivers see only their assigned rides; riders create requests via service-role operations.

**Migrations**: Database evolution tracked in `supabase/migrations/phase*.sql`. Recent focus: Phase 6 (idempotent rides with rate limiting and phone-based identification).

## Key Features & Behavior

### Idempotent Ride Creation

- **Rider Identification**: Server-side stable identifier from phone (priority), client_id (fallback), or IP+UA hash
- **API Endpoint**: `POST /api/rides` checks for existing active ride and returns it instead of creating duplicates
- **Rate Limiting**: 3 requests per 10 minutes per identifier via `rider_rate_limits` table
- **Client Rehydration**: UI stores `ralli_ride_id` in localStorage and restores active ride on page reload
- **GET endpoint**: `GET /api/rides?event_id=...&rider_name=...&rider_phone=...` returns both identifier and any existing active ride

### Ride State Machine

A ride progresses through states: `waiting` → `assigned` → `arrived` → `in_progress` → `completed`. Cancellation can occur at any active state. Transitions are validated in `dispatchService.ts`.

### Smart Dispatch

`dispatchService.ts` implements:
- **Haversine distance calculation** to find nearest available drivers
- **Batch auto-dispatch**: Automatically assigns waiting rides to available drivers based on proximity
- **ETA estimation** for pickup and delivery
- **No-show detection**: Cron job monitors riders who don't confirm presence

### Real-time Updates

Supabase Realtime subscriptions power live queue updates, driver status changes, and ride status transitions without polling.

## Important Patterns & Conventions

### Service Functions

All service functions follow a consistent pattern:
- Accept typed inputs (from `types/database.ts`)
- Return `{ data: T | null, error: Error | null }` or raw data with error handling
- Use `supabaseClient` (public) for client-side operations and `createAdminClient` (service-role) for server-side admin operations

### API Error Handling

- **400**: Missing or invalid required parameters
- **429**: Rate limit exceeded (for `/api/rides`)
- **500**: Server errors (database, Supabase, validation failures)
- **23505**: PostgreSQL unique constraint violation (handled gracefully in `/api/rides` to return existing ride)

### Type Safety

- All database types defined in `types/database.ts` (exported from Supabase)
- API request/response bodies should match service function signatures
- Use `as any` sparingly and only when Supabase SDK lacks precise types

### Supabase Clients

- **`supabaseClient`**: Browser/client-side, uses anon key, respects RLS
- **`createAdminClient()`**: Server-side only, uses service role key, bypasses RLS (for privileged operations)
- Never expose service role key to client-side code

## Testing & Debugging

### Integration Tests

The `npm run test:integration` script validates idempotent ride creation:
```bash
npm run test:integration -- https://myapp.vercel.app <event_id>
```

It submits identical ride payloads twice and verifies the same ride ID is returned both times.

### Local Development

- Supabase local dev mode can be used (requires Docker, managed via CLI)
- `.env.local` controls which Supabase project is targeted
- Real-time updates via Supabase work in development without special setup

## Deployment Notes

- **Vercel**: Recommended deployment platform (seamless Next.js integration)
- **Environment variables**: Ensure all required env vars are set in Vercel project settings before deploying
- **Migrations**: Apply pending migrations in `supabase/migrations/` via Supabase dashboard before deploying corresponding code changes
- **CORS**: Google Maps API key should have appropriate domain restrictions configured in Google Cloud Console

## File Structure Quick Reference

```
lib/
├── supabaseClient.ts        # Browser Supabase client (anon key)
├── supabaseServer.ts        # Server Supabase client (service role key)
└── services/                # Business logic layer
    ├── rideGuardService.ts  # Rider identification & idempotency
    ├── dispatchService.ts   # Auto-dispatch & state machine
    ├── rides.ts
    ├── drivers.ts
    ├── events.ts
    └── ...

app/
├── api/
│   ├── rides/route.ts       # Core idempotent ride API
│   ├── rider/confirm-presence/route.ts
│   └── cron/process-noshow/route.ts
└── {admin,driver,rider}/    # Role-based UIs

supabase/
├── schema.sql               # Initial schema (superseded by migrations)
└── migrations/phase*.sql    # Database evolution

types/database.ts            # TypeScript types from Supabase
```

## Recent Work & Current State

The codebase recently stabilized around idempotent ride creation (Phase 6) with phone-based identification as the primary stable identifier. The rate-limiting and idempotency guards ensure riders cannot create duplicate rides even if they submit multiple times or refresh the page. The system now prioritizes `rider_phone_normalized` for identification, falling back to `client_id`, then IP+UA hash.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Naty89)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Naty89)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
