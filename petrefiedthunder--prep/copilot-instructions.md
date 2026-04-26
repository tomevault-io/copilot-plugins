## prep

> **PrepChef** is a two-sided commercial kitchen rental marketplace connecting culinary entrepreneurs with commercial kitchen space. Think "Airbnb for commercial kitchens." The repo lives at https://github.com/PetrefiedThunder/Prep and deploys to https://prep-ashen.vercel.app.

# CLAUDE.md — PrepChef Autonomous Development Guide

## Project Identity

**PrepChef** is a two-sided commercial kitchen rental marketplace connecting culinary entrepreneurs with commercial kitchen space. Think "Airbnb for commercial kitchens." The repo lives at https://github.com/PetrefiedThunder/Prep and deploys to https://prep-ashen.vercel.app.

**Status**: MVP Complete (Sprint 0–7). Now entering Phase 2 development.

**Founder context**: Solo founder, non-technical visionary. Code quality, security, and production-readiness matter more than speed. Every commit should make the product more trustworthy and closer to market.

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Framework | Next.js 14 (App Router) | Server and client components, server actions |
| Language | TypeScript | Strict mode, type safety everywhere |
| Styling | TailwindCSS | Utility-first, mobile-responsive |
| Database | Supabase (PostgreSQL) | Managed Postgres with Row Level Security |
| Auth | Supabase Auth | Email/password, JWT-based sessions |
| Payments | Stripe Connect | Two-sided marketplace payments, 10% platform fee |
| Hosting | Vercel | Serverless deployment |
| Logging | Custom structured logger | `lib/logger` — JSON format, auto-redacts secrets |

## Repository Structure

```
Prep/
├── app/                    # Next.js App Router pages and API routes
│   └── api/webhooks/stripe/ # Stripe webhook handler (idempotent booking creation)
├── components/             # React components
├── lib/                    # Shared utilities, Supabase clients, logger
├── middleware/              # Custom middleware modules
├── middleware.ts            # Root middleware (auth session refresh)
├── supabase/               # Database migrations and Supabase config
├── tests/                  # Test suite
├── docs/                   # Additional documentation
├── .github/                # CI/CD workflows
├── API.md                  # Server actions and API route documentation
├── SCHEMA.md               # Complete database schema with RLS policies
├── SECURITY.md             # Security posture documentation
├── DEPLOYMENT.md           # Production deployment guide
├── DEPLOYMENT_MEMO.md      # Deployment notes
└── INSTALL.md              # Local development setup
```

## Database Schema (6 tables)

1. **profiles** — Extends auth.users with email, full_name, phone. Auto-created via trigger on signup. RLS: users see/edit only their own profile.
2. **kitchens** — Listings with owner_id (FK → profiles), title, description, address, city, state, zip_code, price_per_hour, max_capacity, square_feet, is_active. RLS: owners manage their own, public can view active listings.
3. **kitchen_photos** — Photo URLs linked to kitchens. RLS: owners manage their own kitchen photos.
4. **stripe_accounts** — Stripe Connect account linking for kitchen owners. RLS: users see only their own.
5. **bookings** — Reservations with payment info, created idempotently via Stripe webhooks. RLS: owners see bookings for their kitchens, renters see their own bookings.
6. **payouts** — Owner payout tracking (90% revenue after 10% platform fee). RLS: owners see only their own payouts.

Full schema documentation: `SCHEMA.md`

## Architecture Pattern

```
Browser → Next.js Pages → Server Actions → Supabase Client (Browser)
                ↓
Vercel Edge: Middleware (Auth Refresh) → Server Actions → API Routes
                ↓                                         ↓
        Supabase Cloud                          Stripe API
     (PostgreSQL + RLS + Auth)       (Connect + Checkout + Webhooks)
```

Key patterns:
- **Server Actions** with `'use server'` for type-safe mutations
- **Middleware** refreshes auth sessions on every request
- **Stripe webhooks** create bookings idempotently (prevents duplicates)
- **RLS everywhere** — database-enforced access control, not application-level

## Phase 2 Roadmap (ACTIVE — Work on these)

These are the founder's defined next priorities. Work through them in order, one feature at a time. Each feature should be a complete, tested, production-ready implementation on its own branch before merging to main.

### Priority 1: Photo Upload to Supabase Storage
- Replace URL-based kitchen photos with actual file uploads
- Use Supabase Storage with signed URLs
- Add image optimization/resizing
- Update kitchen creation and edit flows
- Add drag-and-drop upload UI

### Priority 2: Calendar Availability Management
- Kitchen owners define available time slots
- Prevent double-booking at the database level
- Calendar UI for viewing and managing availability
- Integrate availability checking into the booking flow

### Priority 3: Email Notifications
- Booking confirmation emails (to both owner and renter)
- Booking reminder emails (24 hours before)
- New booking alerts for kitchen owners
- Use a service like Resend or Supabase Edge Functions

### Priority 4: Reviews and Ratings
- Renters can review kitchens after completed bookings
- Star ratings (1–5) with text reviews
- Average rating displayed on kitchen listings
- New database table with RLS policies
- Prevent reviews before booking completion

### Priority 5: Advanced Search Filters
- Filter by amenities/equipment
- Filter by availability dates
- Sort by rating, price, distance
- Add amenities/equipment fields to kitchen schema

### Priority 6: Mobile App Considerations
- Ensure all pages are fully responsive
- Add PWA manifest and service worker
- Optimize touch interactions
- Test on actual mobile devices

## Phase 3 Roadmap (FUTURE — Do not start until Phase 2 is complete)
- Multi-city expansion
- Kitchen verification process
- Insurance integration
- Analytics dashboard
- Referral program

## Development Standards

### Code Quality
- TypeScript strict mode — no `any` types, no `@ts-ignore`
- All new code must pass `npm run build` (type checking + compilation)
- All new code must pass `npm run lint` (ESLint)
- Use the existing structured logger (`lib/logger`) — never use raw `console.log`
- Follow existing patterns in the codebase — consistency over novelty

### Git Workflow
- Create a feature branch for each piece of work (e.g., `feature/photo-upload`, `feature/calendar-availability`)
- Write clear, descriptive commit messages
- Each branch should be independently deployable
- Never commit secrets, API keys, or credentials

### Testing
- Run `npm run build` after every significant change
- Run `npm run lint` to check for issues
- Run `npm test` for existing tests
- Add tests for new functionality, especially for:
  - Server actions (happy path + error cases)
  - Webhook handlers
  - RLS policy behavior
  - Edge cases in booking/payment flows

### Security
- All new tables MUST have RLS policies
- Validate all user input server-side
- Use parameterized queries (Supabase client handles this)
- Verify Stripe webhook signatures
- Never expose secrets in client-side code
- Redact sensitive data in logs using the existing logger

### Database Changes
- All schema changes go through Supabase migrations in `supabase/`
- Document new tables/columns in `SCHEMA.md`
- Always add RLS policies for new tables
- Test RLS policies to verify they enforce the correct access patterns

### UI/UX
- Mobile-first responsive design with TailwindCSS
- Consistent with existing design patterns in the app
- Accessible (proper labels, focus management, semantic HTML)
- Loading states for async operations
- Error states with helpful messages

## What NOT to Do
- **Do not refactor the entire codebase** — make targeted, incremental improvements
- **Do not add unnecessary dependencies** — the lean approach (15 deps) is intentional
- **Do not add complex infrastructure** (Kubernetes, Docker, microservices, ETL pipelines) — this was deliberately removed in the rebuild
- **Do not add AI integrations** — not in scope for Phase 2
- **Do not change the payment flow** without extreme care — Stripe Connect with webhooks is working and tested
- **Do not break existing functionality** — run the build and tests before considering work complete
- **Do not commit to main directly** — always use feature branches
- **Do not skip RLS policies** on any new table
- **Do not use `console.log`** — use the structured logger

## Autonomous Work Cycle

When running autonomously, follow this cycle for each work session:

1. **Orient**: Pull latest from main. Run `npm run build` and `npm test` to confirm the repo is healthy.
2. **Plan**: Identify the next item on the Phase 2 roadmap that hasn't been completed. If all Phase 2 items are done, move to Phase 3.
3. **Branch**: Create a feature branch from main.
4. **Implement**: Build the feature incrementally. Commit frequently with clear messages.
5. **Verify**: Run `npm run build`, `npm run lint`, and `npm test`. Fix any failures.
6. **Document**: Update `SCHEMA.md` if schema changed. Update `API.md` if new routes/actions added. Add inline code comments for complex logic.
7. **PR**: Create a pull request with a clear description of what was built and why.
8. **Repeat**: Move to the next roadmap item.

If a build or test fails and you cannot resolve it after 3 attempts, stop work on that feature, document the issue in the PR description, and move to the next roadmap item.

## Environment Variables Required

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

These are set in `.env.local` for development and in Vercel for production. Never commit them.

## Key Files to Understand Before Making Changes

- `middleware.ts` — Auth session refresh on every request
- `app/api/webhooks/stripe/route.ts` — Stripe webhook handler (critical payment flow)
- `lib/logger.ts` — Structured logging with secret redaction
- `lib/supabase/` — Supabase client configuration (browser and server)
- `supabase/` — Database migrations and schema

## Project Metrics (MVP Baseline)

- ~3,500 lines of code
- ~40 files
- 15 dependencies
- <30 second build time
- 1,458 commits on main

Track these as development continues. The goal is controlled growth, not bloat.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PetrefiedThunder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
