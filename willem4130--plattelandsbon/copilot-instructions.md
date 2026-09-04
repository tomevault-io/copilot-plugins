## plattelandsbon

> Dutch rural tourism voucher web app — consumer browse/claim + business management + admin approval.

# Plattelandsbon (Oom Gerrit)

Dutch rural tourism voucher web app — consumer browse/claim + business management + admin approval.
Part of the "Oom Gerrit" brand. This is a standalone git repo at `~/Dev/oomgerrit/plattelandsbon/`.

## Architecture: Onion Architecture (Strict)

ALL code follows onion architecture with strict modularity. Dependencies ALWAYS point inward.

```
Presentation (app/, components/) → Server (tRPC routers) → Application (use cases) → Domain (entities)
                                                                    ↑
                                          Infrastructure (Prisma repos, AI, email, scraping)
```

### Layers

**Domain** (`src/domain/`) — ZERO external dependencies
- `entities/` — Business objects with protected constructors + `static create()`/`fromProps()` factories
- `value-objects/` — Immutable typed values (ClaimCode, VoucherStatus, DiscountType)
- `repositories/` — INTERFACES only (IVoucherRepository, IClaimRepository) — all methods accept optional `tx?: TransactionContext`
- `types/` — TransactionContext (opaque), PaginationOptions, ITransactionManager
- `services/` — Pure domain logic (VoucherValidationService, FraudDetectionService)
- `errors/` — Domain-specific errors

**Application** (`src/application/`) — Orchestrates domain, no framework deps
- `interfaces/` — `IUseCase<TInput, TOutput>` generic interface
- `use-cases/` — One class per operation, all implement `IUseCase`
- `dtos/` — Data transfer objects for layer boundaries
- `mappers/` — Entity <-> DTO conversions

**Infrastructure** (`src/infrastructure/`) — Framework/external deps live here
- `repositories/` — Prisma implementations extending `BaseRepository<TEntity, TPrismaRecord>`
- `services/PrismaTransactionManager.ts` — implements `ITransactionManager` for cross-repo transactions
- `services/ai/` — Anthropic API integration
- `services/email/` — Resend + react-email templates
- `services/scraping/` — Competitor scrapers (Groupon, SocialDeal, WeekendjeWeg)
- `services/storage/` — Vercel Blob
- `services/cache/` — Upstash Redis
- `config/container.ts` — Dependency wiring (manual factory functions)

**Server** (`src/server/api/`) — Thin tRPC layer
- Routers call use cases, nothing else
- One router per domain (vouchers, claims, businesses, categories, users, auth)

**Presentation** (`src/app/`, `src/components/`)
- Next.js App Router pages — render only
- React components — no business logic

### Key Patterns

**Entity creation:**
```typescript
// New entity (use cases/domain services):
const business = Business.create(props)
// Reconstitution from DB (infrastructure repos):
const business = Business.fromProps(props)
// Direct `new Entity()` is forbidden (protected constructor)
```

**Use case signature:**
```typescript
class RegisterBusinessUseCase implements IUseCase<BusinessRegistrationDTO, BusinessResponseDTO> {
  constructor(private businessRepo: IBusinessRepository) {}
  async execute(input: BusinessRegistrationDTO): Promise<BusinessResponseDTO> { ... }
}
```

**Repository with transactions:**
```typescript
// All repo methods accept optional tx:
findById(id: string, tx?: TransactionContext): Promise<Entity | null>
// Transaction usage:
await transactionManager.run(async (tx) => {
  await voucherRepo.incrementClaimCount(voucherId, tx)
  await claimRepo.create(claimData, tx)
})
```

**BaseRepository (infrastructure):**
```typescript
class PrismaBusinessRepo extends BaseRepository<Business, PrismaRecord> implements IBusinessRepository {
  protected toDomain(record: PrismaRecord): Business { return Business.fromProps(...) }
  // Uses this.getClient(tx), this.mapOrNull(), this.mapMany()
}
```

### Rules
- Domain layer imports NOTHING from other layers
- Entities have protected constructors — use `create()` or `fromProps()`
- All use cases implement `IUseCase<TInput, TOutput>`
- All Prisma repos extend `BaseRepository` and accept `tx?: TransactionContext`
- tRPC routers are thin: validate input, call use case, return result
- Prisma is NEVER imported outside `src/infrastructure/`
- Use cases depend on repository interfaces, not implementations
- Components never call Prisma or use cases directly

## Tech Stack

Next.js 16, React 19, TypeScript 5.9, PostgreSQL, Prisma 6, NextAuth v5, tRPC v11, Tailwind CSS 4, shadcn/ui, GSAP + @gsap/react (hero scroll animations), Anthropic AI SDK, Resend, Vercel Blob, Upstash Redis

See `RESEARCH.md` for validated versions and configuration details.

## File Organization

- tRPC routers → `src/server/api/routers/`, one per domain
- Use cases → `src/application/use-cases/`, grouped by domain
- Use case interface → `src/application/interfaces/IUseCase.ts`
- Domain entities → `src/domain/entities/` (Business, Voucher, VoucherClaim, Category, User)
- Domain types → `src/domain/types/` (TransactionContext, PaginationOptions)
- Value objects → `src/domain/value-objects/` (ClaimStatus, VoucherStatus, DiscountType, UserRole, BusinessStatus)
- Repository interfaces → `src/domain/repositories/` (IVoucherRepository, IClaimRepository, IBusinessRepository, IUserRepository, ICategoryRepository)
- Repository implementations → `src/infrastructure/repositories/`
- Base repository → `src/infrastructure/repositories/BaseRepository.ts`
- DI container → `src/infrastructure/config/container.ts`
- Components → `src/components/`, organized by feature
- UI components → `src/components/ui/` (shadcn)
- Shared utilities → `src/lib/`
- Static images → `public/images/`
- Seed data → `prisma/seed-data/achterhoek.ts`
- Database schema → `prisma/schema.prisma`

### Landing Page Components

Modular components in `src/components/landing/`:
- `navbar.tsx` — Sticky nav + mobile menu (uses navLinks from constants, UserMenu for auth)
- `user-menu.tsx` — Auth-aware navbar component (login button or user dropdown with role-based links)
- `hero-section.tsx` — GSAP-animated hero (self-contained useRef + useGSAP)
- `how-it-works.tsx` — 3-step journey timeline with connecting lines
- `featured-vouchers.tsx` — Voucher carousel (CSS scroll-snap) with category filters
- `featured-businesses.tsx` — Business carousel (CSS scroll-snap) with category filters
- `landing-data.tsx` — Client component wrapper for data-dependent sections (uses useQuery, data pre-hydrated from server)
- `all-vouchers.tsx` — Full voucher grid (used on /bonnen page)
- `business-cta.tsx` — Entrepreneur call-to-action section
- `footer.tsx` — Footer with nav links
- `voucher-card.tsx` — Reusable voucher card (carousel + grid)
- `business-card.tsx` — Reusable business card (carousel + grid)
- `constants.ts` — Shared categoryMeta, navLinks, getDiscountLabel
- `types.ts` — VoucherItem, BusinessItem (inferred from tRPC router output)

### Intercepting Routes (Modal System)

Clicking a voucher/business card opens a modal overlay instead of navigating away. Uses Next.js parallel routes + intercepting routes:
- `src/app/@modal/default.tsx` — null default for parallel route slot
- `src/app/@modal/(.)bon/[id]/page.tsx` — intercepts `/bon/[id]`, renders modal
- `src/app/@modal/(.)bedrijf/[id]/page.tsx` — intercepts `/bedrijf/[id]`, renders modal
- `src/components/modal/route-modal.tsx` — Reusable modal shell (Radix Dialog, prev/next arrows, keyboard navigation)
- `src/components/modal/voucher-modal-content.tsx` — Voucher detail in modal (uses placeholderData from list cache for instant render)
- `src/components/modal/business-modal-content.tsx` — Business detail in modal

Direct visits to `/bon/[id]` or `/bedrijf/[id]` render the full page (SEO, sharing).

### Server-Side Prefetching

Landing page (`src/app/page.tsx`) is a Server Component that prefetches tRPC data:
- Uses `createHydrationHelpers` from `@trpc/react-query/rsc` (configured in `src/trpc/server.ts`)
- Exports `trpc` (with `.prefetch()`) and `HydrateClient` wrapper
- Client components get pre-hydrated data — no loading flash on first render

### Page Routes

- `/` — Landing page (hero, how it works, featured carousels, business CTA)
- `/bonnen` — All vouchers with category filters
- `/bedrijven` — All businesses with category + city filters
- `/voor-ondernemers` — Business info/landing page (benefits, how it works, naoberschap)
- `/bon/[id]` — Voucher detail page (full page for direct visits + claim flow)
- `/bedrijf/[id]` — Business detail page (full page for direct visits)
- `/inloggen` — Login page (tabs: credentials, magic link + Google button)
- `/inloggen/verify-request` — Magic link "check your email" page
- `/registreren` — Consumer registration (auto sign-in after register)
- `/register/business` — Business registration form
- `/business/vouchers` — Business voucher management
- `/business/vouchers/create` — Create voucher form
- `/admin/businesses` — Admin business verification
- `/admin/users` — Admin user management
- `/admin/vouchers/pending` — Admin voucher approval

### Modularity Principles
- Single responsibility per file
- Clear, descriptive file names
- Group related functionality by domain
- No monolithic files
- Every new feature must respect layer boundaries

## Code Quality — Zero Tolerance

After editing ANY file, run:

```bash
npm run lint
npm run typecheck
```

Fix ALL errors/warnings before continuing.

If changes require server restart:
1. Restart: `npm run dev`
2. Read server output/logs
3. Fix ALL warnings/errors

## Design System

**Brand colors** (globals.css CSS variables):
- Primary: Meadow green `HSL(142, 45%, 32%)` — nature/rural
- Accent: Harvest amber `HSL(36, 80%, 94%)` — warmth/gezellig

**CSS utilities** (globals.css `@layer utilities`):
- `.glass` — Glassmorphism: `bg-white/70 backdrop-blur-md border-white/30 shadow-xl`
- `.glass-subtle` — Lighter glass variant
- `.glossy-card` — Glass card with hover shadow transition
- `.glossy-btn` — Gradient white button with shadow

**Carousels** use native CSS scroll-snap (`overflow-x-auto snap-x snap-mandatory`). No JS carousel library — browser handles scroll, swipe, trackpad natively. Arrow buttons call `scrollBy()`.

**Animations** (GSAP + ScrollTrigger):
- Hero section uses `useGSAP` hook with `gsap.fromTo()` + `autoAlpha` for scroll reveals
- Hero image reveal triggered via `onLoad` callback (waits for image to load)
- Guard `gsap.registerPlugin()` with `typeof window !== 'undefined'` for SSR
- Use `data-*` attributes for animation targets (not CSS classes — avoids conflicts)
- Use `autoAlpha` instead of `opacity` (handles `visibility` too)
- Avoid `transition-all` on animated elements — use specific properties (`transition-shadow`, `transition-transform`)

**Images**: Unsplash (free commercial license), stored in `public/images/`

## Local Development

### Prerequisites
- Docker Desktop running
- Node.js 20.9+

### First-time setup
```bash
npm install
npm run db:up              # start PostgreSQL in Docker
npm run db:migrate         # create tables
npm run db:seed            # populate test data
npm run dev                # start Next.js (localhost:3000)
```

### Daily development
```bash
npm run db:up              # start DB (if not already running)
npm run dev                # start app
```

### Useful commands
```bash
npm run db:studio          # browse data at localhost:5555
npm run db:reset           # nuke DB, re-migrate, re-seed
npm run db:down            # stop PostgreSQL container
docker compose down -v     # stop + delete all data (fresh start)
```

## Database Management — Local vs Production

Local (Docker) and production (Neon) are **completely separate**. Never cross them.

### Schema changes (migrations)
```
Local:  edit schema.prisma → npm run db:migrate → creates migration file → commit to git
                                                                               ↓
Production:  deploy to Vercel → build runs prisma migrate deploy → applies migration
```

### Rules
- **Never run `prisma migrate dev` against production** — it can reset data
- **Never seed production** — `db:seed` is for test data only
- **Migration files are the bridge** — created locally, applied in production via `prisma migrate deploy`
- `.env` has local `DATABASE_URL` (localhost), Vercel env vars have Neon URL — they never cross
- The `build` script runs `prisma generate && prisma migrate deploy && next build` so Vercel auto-applies migrations on deploy
- Local data is disposable — reset freely with `npm run db:reset`
- Production data is sacred — only modified by the app or explicit migrations

### Environment separation
| | Local | Production |
|---|---|---|
| Database | Docker PostgreSQL 17 | Neon (EU Frankfurt) |
| DATABASE_URL | `postgresql://postgres:postgres@localhost:5432/oomgerrit` | Set in Vercel env vars |
| Migrations | `prisma migrate dev` (creates files) | `prisma migrate deploy` (applies files) |
| Seed data | `npm run db:seed` (test data) | Never — real user data only |
| Reset | `npm run db:reset` (safe, local only) | Never |

## Seed Data

Realistic Achterhoek businesses in `prisma/seed-data/achterhoek.ts`:
- 16 real businesses (restaurants, bars, wellness, accommodations, activities)
- 47 vouchers with researched pricing from actual menus/websites
- Cities: Winterswijk, Doetinchem, Zutphen, Groenlo, Ruurlo, Haarlo, Braamt, Laren
- Re-seed: `npx tsx prisma/seed.ts` (clears and re-creates all data)

## Authentication

NextAuth v5 with JWT session strategy. Three sign-in methods:

1. **Credentials** (email + password) — bcrypt hashing, `ValidateCredentialsUseCase`
2. **Magic link** (email) — Resend + react-email Dutch template, `VerificationToken` model
3. **Google OAuth** — conditional, only active when `AUTH_GOOGLE_ID`/`AUTH_GOOGLE_SECRET` are set

### Key files
- `src/auth.ts` — NextAuth config (providers, JWT/session callbacks, PrismaAdapter)
- `src/middleware.ts` — Protects `/admin/*`, `/business/*`; redirects to `/inloggen`
- `src/types/next-auth.d.ts` — Session + JWT type augmentation (user.id, user.role)
- `src/infrastructure/services/email/resend.ts` — Resend client (guarded, null if no API key)
- `src/infrastructure/services/email/magic-link-email.tsx` — Magic link email template

### tRPC procedure protection
- `publicProcedure` — No auth required (browse, claim, register)
- `protectedProcedure` — Requires authenticated session
- `businessProcedure` — Requires BUSINESS role
- `adminProcedure` — Requires ADMIN role
- Role checks ALWAYS apply (no dev bypass)

### Dev auth bypass
In `NODE_ENV=development`, tRPC context uses `getDevSession()` instead of `auth()`:
- Auto-logs in as first BUSINESS user from DB
- Override with `x-user-id` header for specific user
- Middleware allows all routes in dev (protection is in tRPC layer)
- Role checks still enforced — dev session has a real role

## Branding

- **Product name**: Plattelandsbon (used in navbar, tab title, favicon)
- **Parent brand**: Oom Gerrit
- **Tagline**: "De beste tips van 't platteland"
- **Region**: Achterhoek (Gelderland), not generic "platteland"
- **Tone**: Warm, gezellig, "je/jij" — never corporate

## Deployment Pipeline

**GitHub → Vercel auto-deploy**. No CI/CD config files — Vercel's GitHub integration handles it.

```
git push origin main → Vercel detects push → runs `npm run build` → deploys to production
                                                    ↓
                                    prisma generate → prisma migrate deploy → next build
```

- **Production URL**: https://plattelandsbon.vercel.app
- **Preview deploys**: Every PR/branch push gets a preview URL (`plattelandsbon-<hash>-willem4130s-projects.vercel.app`)
- **Build command**: `npm run build` (= `prisma generate && prisma migrate deploy && next build`)
- **Node.js**: 24.x (set in Vercel project settings)
- **Region**: iad1 (US East) for serverless functions
- **Database**: Neon Postgres (EU Frankfurt) — connection strings in Vercel env vars
- **`.vercelignore`**: Excludes `.env`, `.env.*`, `debug/`

### Deploy checklist
1. Run `npm run lint && npm run typecheck` locally
2. Commit and push to `main`
3. Vercel auto-deploys — check build logs in Vercel dashboard if needed

### Vercel env vars (production)
Managed in Vercel dashboard — never in code. Key vars:
- `DATABASE_URL` / `DIRECT_URL` — Neon connection strings
- `NEXTAUTH_SECRET` — Auth secret (min 32 chars)
- `AUTH_GOOGLE_ID` / `AUTH_GOOGLE_SECRET` — Google OAuth (optional, Google sign-in disabled without these)
- `RESEND_API_KEY` — Email sending (optional, magic links log to console without this)
- `EMAIL_FROM` — Resend sender address

## Project Status

**GitHub**: https://github.com/willem4130/plattelandsbon
**Vercel project**: `plattelandsbon` (owner: willem4130s-projects)
**Production**: https://plattelandsbon.vercel.app
**Current Phase**: Week 3+ — landing page, detail pages, full auth (3 providers), wiring check fixes done (March 2026)
**References**: `PROJECT_PLAN.md` (implementation roadmap), `RESEARCH.md` (validated stack)

---
> Source: [willem4130/plattelandsbon](https://github.com/willem4130/plattelandsbon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
