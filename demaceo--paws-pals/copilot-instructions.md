## paws-pals

> This is a **Next.js 16 dog adoption platform** with a protected admin dashboard. The public site displays adoptable dogs with i18n support (English/Spanish), while admins manage dog profiles through a secure CMS.

# AI Coding Agent Instructions - Dog Adoption Platform

## Project Overview

This is a **Next.js 16 dog adoption platform** with a protected admin dashboard. The public site displays adoptable dogs with i18n support (English/Spanish), while admins manage dog profiles through a secure CMS.

**Stack**: Next.js 16 (App Router) • React 19 • TypeScript • Prisma • PostgreSQL • NextAuth.js 5 • Cloudinary • Tailwind CSS 4

## Critical Architecture Patterns

### Authentication Flow (NextAuth.js 5 Beta)

- **Middleware protection**: [proxy.ts](../proxy.ts) uses NextAuth's `auth` as default export with matcher for `/admin/*` routes (excludes `/admin/login`)
- **Credentials provider**: [auth.config.ts](../auth.config.ts) validates against `Admin` table using bcrypt
- **Session-based**: All admin API routes check `await auth()` before mutations
- **Login redirect**: Unauthorized admin routes → `/admin/login` (configured in `authConfig.pages`)

### Database Patterns (Prisma)

- **JSON string storage**: `Dog.gallery` is stored as `String?` in Prisma but handled as `string[]` in TypeScript
  - Parse on read: `JSON.parse(dog.gallery)` in [lib/dogs.ts](../lib/dogs.ts)
  - Stringify on write: `JSON.stringify(gallery)` in [app/api/dogs/route.ts](../app/api/dogs/route.ts)
- **Status filtering**: Public queries always filter `WHERE status != 'Adopted'` (see `getDogs()`)
- **Client singleton**: Use `prisma` from [lib/prisma.ts](../lib/prisma.ts) - prevents hot reload connection issues

### Image Upload Flow (Cloudinary)

1. Admin selects image via [DogForm](../app/admin/components/DogForm.tsx) → shows crop modal
2. [ImageCropModal](../app/admin/components/ImageCropModal.tsx) allows resize, crop, rotate, and quality adjustment
3. Cropped image converted to blob → `POST /api/upload`
4. [route.ts](../app/api/upload/route.ts) converts File to Buffer → uploads to Cloudinary under `dogs/[dogName]/` folder
5. Returns Cloudinary URL (e.g., `https://res.cloudinary.com/.../dogs/buddy/img.webp`)
6. URLs stored in database, not files in `/public`

- **No local file storage**: All images go to Cloudinary CDN, never commit images to Git
- **Client-side cropping**: Uses `react-easy-crop` library for interactive image editing
- **Image utilities**: [lib/image-utils.ts](../lib/image-utils.ts) handles cropping, rotation, and resizing
- **Automatic resizing**: Default max width 1200px, quality 90% (adjustable in crop modal)

### i18n Implementation (Cookie-Based)

- **Server-side locale detection**: [lib/i18n-server.ts](../lib/i18n-server.ts) checks cookie → `Accept-Language` header → defaults to "en"
- **Message structure**: Flat JSON in [lib/i18n-messages.ts](../lib/i18n-messages.ts) with dot notation keys (e.g., `"hero.title"`)
- **Client hydration**: [LocaleProvider](../app/i18n/LocaleProvider.tsx) wraps app with React Context for `useI18n()` hook
- **Switching**: [LanguageSwitcher](../app/components/LanguageSwitcher.tsx) calls server action `setLocaleCookie()` → revalidates path

### Validation Strategy (Zod)

- **Shared schemas**: [lib/validations.ts](../lib/validations.ts) exports `dogSchema`, `dogUpdateSchema`, `loginSchema`
- **Used in**: API routes (`.parse()` throws 400), form submissions, TypeScript types via `z.infer<>`
- **Enum enforcement**: `sex`, `size`, `status` use Zod enums matching Prisma schema exactly

## Development Workflows

### Local Development Setup

```bash
pnpm install                          # Install dependencies
npx prisma migrate dev --name init    # Create database + run migrations
npx prisma generate                   # Generate Prisma Client
pnpm dev                              # Start dev server (localhost:3000)
```

### Database Operations

```bash
npx prisma studio                     # Open database GUI (localhost:5555)
npx prisma migrate dev --name <name>  # Create new migration
npx prisma db seed                    # Run seed file (seeds default admin + 9 dogs)
npx prisma migrate reset              # ⚠️ Nuke database and re-seed
```

### Admin Credentials

- **Default credentials** (must change before deployment):
  - Email: `admin@example.com`
  - Password: `password123`
- Set in `.env`: `ADMIN_EMAIL` and `ADMIN_PASSWORD` (bcrypt hashed in seed)
- **⚠️ SECURITY**: Always change default credentials before deploying to production
- Seed script ([prisma/seed.ts](../prisma/seed.ts)) creates admin user from env vars

## Documentation Maintenance

**CRITICAL**: When adding, modifying, or removing features, AI agents MUST update relevant documentation:

### Always Update When:

- **Adding new features**: Update README.md, add to Feature Roadmap section below
- **Changing architecture**: Update this file and [documentation/ARCHITECTURE.md](../documentation/ARCHITECTURE.md)
- **Modifying API routes**: Document in [documentation/ARCHITECTURE.md](../documentation/ARCHITECTURE.md) data flow sections
- **Adding database models**: Update [prisma/schema.prisma](../prisma/schema.prisma) and document in Database Patterns section above
- **Changing environment variables**: Update deployment section and create/update `.env.example`
- **Adding dependencies**: Document integration patterns in Integration Points section
- **Modifying authentication**: Update Authentication Flow section above
- **Changing testing approach**: Update Testing Strategy section below

### Documentation Files to Consider:

```
README.md                              # User-facing overview, features, quick start
.github/copilot-instructions.md        # This file - architecture, workflows, patterns
documentation/ARCHITECTURE.md          # System architecture, data flows
documentation/TESTING_CHECKLIST.md     # Manual testing procedures (update for new features)
documentation/ADMIN_SETUP.md           # Admin dashboard setup guide
documentation/DATABASE_SETUP.md        # Database configuration options
QUICKSTART.md                          # Quick reference commands
```

### Examples:

- ✅ **Added inquiry submission API** → Update ARCHITECTURE.md data flows + Feature Roadmap + README.md features
- ✅ **Changed gallery field to PostgreSQL array** → Update Database Patterns section + Common Gotchas
- ✅ **Added Stripe integration** → Update Integration Points + README.md dependencies + Feature Roadmap
- ✅ **Implemented automated tests** → Update Testing Strategy section + add test documentation file

## Project-Specific Conventions

### File Structure Patterns

- **Server Components by default**: All components in `app/` are server unless marked `"use client"`
- **API routes**: Follow Next.js 16 conventions - `route.ts` files export named HTTP methods
- **Admin components**: Live in [app/admin/components/](../app/admin/components/), not shared `app/components/`
- **Route groups**: `(protected)` group in [app/admin/](../app/admin/) shares layout without affecting URLs

### Styling Approach

- **Tailwind utility-first**: No CSS modules, all styles inline with Tailwind classes
- **Shared style objects**: [lib/styles.ts](../lib/styles.ts) exports `buttonStyles`, `cardStyles`, `inputStyles` for consistency
- **Dark mode**: Automatic via `ThemeProvider` + `dark:` classes (uses system preference + toggle)
- **Responsive breakpoints**: Mobile-first with `md:` (768px), `lg:` (1024px) prefixes

### State Management Patterns

- **Filter persistence**: [FilterPersistence.tsx](../app/components/FilterPersistence.tsx) saves to localStorage, not URL params
- **Server Actions**: Use for i18n switching (see `setLocaleCookie` in `i18n-server.ts`), not form submissions
- **No global state library**: React Context for i18n/theme only, forms use local state

### TypeScript Patterns

- **Strict null checks**: Enabled in [tsconfig.json](../tsconfig.json)
- **Type imports**: Always use `import type` for types (e.g., `import type { Dog } from "@/lib/dogs"`)
- **Path aliases**: `@/` maps to project root (configured in tsconfig)

## Integration Points

### Cloudinary

- **Configuration**: Auto-configures from `CLOUDINARY_URL` or individual env vars in [upload/route.ts](../app/api/upload/route.ts)
- **Transformations**: URLs support query params for resizing (e.g., `?w=800&q=auto`)
- **Folder structure**: All dog images under `dogs/[dogName]/` prefix in Cloudinary

### Prisma Migrations

- **PostgreSQL in production**: Schema uses `provider = "postgresql"` (not SQLite)
- **Migration files**: Hand-editable in [prisma/migrations/](../prisma/migrations/)
- **Schema changes**: Always run `prisma migrate dev` after editing [schema.prisma](../prisma/schema.prisma)

### NextAuth.js 5 Beta

- **Different from v4**: Uses `auth()` function directly, not `getServerSession(authOptions)`
- **Middleware syntax**: Export `auth` as default, not wrapped in `withAuth()`
- **Beta gotchas**: May have breaking changes - pin to exact version `5.0.0-beta.30`

## Common Gotchas

1. **Gallery field**: Always remember JSON.parse/stringify - it's a string in DB, array in code
2. **Status field**: Never show adopted dogs publicly - all public queries filter `status != 'Adopted'`
3. **Auth check timing**: API routes must `await auth()` before accessing request body
4. **Image paths**: Don't reference `/public/dogs/` - all images are Cloudinary URLs now
5. **Locale detection**: Runs on every request - don't call `getLocale()` in client components
6. **Prisma Client generation**: Run `npx prisma generate` after schema changes or you'll get import errors
7. **Middleware matcher**: Carefully exclude `/admin/login` or you'll create redirect loops

## Deployment (Vercel)

### Configuration

- **Auto-detection**: Vercel automatically detects Next.js and configures build settings
- **Deployment trigger**: Automatic on `git push` to main branch
- **Preview deployments**: Enabled for all branches (automatic staging environments)

### Environment Variables

All environment variables are pre-configured in Vercel dashboard. Critical variables for production:

```bash
DATABASE_URL=<production-postgresql-url>     # Supabase production connection string
NEXTAUTH_URL=https://paws-pals-xi.vercel.app # Must match production domain
NEXTAUTH_SECRET=<strong-random-secret>       # Generate with: openssl rand -base64 32
CLOUDINARY_CLOUD_NAME=<your-cloud-name>      # Same as local
CLOUDINARY_API_KEY=<your-api-key>            # Same as local
CLOUDINARY_API_SECRET=<your-api-secret>      # Same as local
ADMIN_EMAIL=<production-admin-email>         # Real email (not admin@example.com)
ADMIN_PASSWORD=<strong-password>             # Strong password (not password123)
```

### Deployment Checklist

- ✅ Update `NEXTAUTH_URL` to production domain (not `localhost:3000`)
- ✅ Generate new `NEXTAUTH_SECRET` for production (never reuse local secret)
- ✅ Change default admin credentials from `admin@example.com` / `password123`
- ✅ Run database migrations on production database (if not automated)
- ✅ Verify Cloudinary credentials are configured
- ✅ Test admin login after deployment

## Testing Strategy

### Current State

- **Manual testing**: Comprehensive checklist in [documentation/TESTING_CHECKLIST.md](../documentation/TESTING_CHECKLIST.md)
- **No automated tests yet**: Project currently relies on manual QA

### Planned Testing Suite (AI Agents Should Build)

When implementing automated tests, focus on:

1. **Admin workflow testing** (highest priority):
   - Authentication flow (login, logout, session persistence)
   - CRUD operations (create, read, update, delete dogs)
   - Image upload validation (file types, size limits, Cloudinary integration)
   - Form validation (Zod schema enforcement)
   - Authorization checks (API routes reject unauthenticated requests)

2. **Recommended testing stack**:
   - **Unit tests**: Vitest for lib functions (`lib/dogs.ts`, `lib/validations.ts`, `lib/i18n-messages.ts`)
   - **Integration tests**: Vitest + MSW for API routes (`app/api/**`)
   - **E2E tests**: Playwright for user workflows (admin dashboard, public site)

3. **Test patterns to follow**:
   - Mock Prisma client using `prisma-mock` or in-memory SQLite
   - Mock Cloudinary uploads in tests (don't hit real API)
   - Use test database URL: `DATABASE_URL_TEST` in `.env.test`
   - Seed test data before each test suite

4. **Coverage goals**:
   - Admin API routes: 90%+ coverage (security-critical)
   - Database queries: 80%+ coverage
   - Validation schemas: 100% coverage (Zod schemas)
   - Public API routes: 70%+ coverage

### Example Test Structure

```
__tests__/
├── unit/
│   ├── lib/dogs.test.ts              # getDogs(), getDog(), parsing logic
│   ├── lib/validations.test.ts       # Zod schema validation
│   └── lib/i18n-messages.test.ts     # Translation formatting
├── integration/
│   ├── api/dogs.test.ts              # CRUD endpoints, auth checks
│   ├── api/upload.test.ts            # Image upload, file validation
│   └── api/auth.test.ts              # NextAuth credential provider
└── e2e/
    ├── admin-auth.spec.ts            # Login, logout, session
    ├── admin-crud.spec.ts            # Full dog management workflow
    ├── admin-upload.spec.ts          # Image upload UI flow
    └── public-site.spec.ts           # Browse dogs, filters, i18n
```

## Feature Roadmap

### Planned Features (Consider in Architecture Decisions)

When building new features or refactoring, keep these future additions in mind:

1. **Visitor inquiry submissions** (near-term):
   - Admin receives adoption inquiry emails (not just form submissions)
   - Store inquiries in database for tracking
   - Add `Inquiry` model to Prisma schema

2. **Payment processing for donations**:
   - Stripe integration for one-time/recurring donations
   - Donation tracking and receipts
   - Public-facing donation page

3. **Multiple admin roles**:
   - Role-based access control (viewer, editor, super-admin)
   - Update `Admin` model with `role` field
   - Middleware checks for role-specific permissions

4. **Email subscription newsletters**:
   - Newsletter signup form on public site
   - SendGrid/Mailchimp integration
   - Subscriber management in admin dashboard

5. **Volunteer request system**:
   - Public volunteer application form
   - Admin review/approval workflow
   - Volunteer database and scheduling

### Architectural Implications

- **Email service**: Plan to integrate SendGrid or similar (add email templates, queuing)
- **Payment handling**: Design for PCI compliance (Stripe Checkout, no raw card data)
- **RBAC system**: Admin API routes will need role checks (not just auth checks)
- **Form submissions**: Consider shared form handling pattern (inquiries, volunteers, donations)

## Common Gotchas

1. **Gallery field**: Always remember JSON.parse/stringify - it's a string in DB, array in code
2. **Status field**: Never show adopted dogs publicly - all public queries filter `status != 'Adopted'`
3. **Auth check timing**: API routes must `await auth()` before accessing request body
4. **Image paths**: Don't reference `/public/dogs/` - all images are Cloudinary URLs now
5. **Locale detection**: Runs on every request - don't call `getLocale()` in client components
6. **Prisma Client generation**: Run `npx prisma generate` after schema changes or you'll get import errors
7. **Middleware matcher**: Carefully exclude `/admin/login` or you'll create redirect loops

## Testing Checklist

See [documentation/TESTING_CHECKLIST.md](../documentation/TESTING_CHECKLIST.md) for comprehensive manual testing guide covering:

- Public dog browsing (filters, detail pages)
- Admin CRUD operations
- Image uploads
- Authentication flows
- i18n switching

## Key Files Reference

- **Data layer**: [lib/dogs.ts](../lib/dogs.ts) - all database queries
- **API routes**: [app/api/dogs/route.ts](../app/api/dogs/route.ts), [app/api/dogs/[id]/route.ts](../app/api/dogs/[id]/route.ts)
- **Auth config**: [auth.config.ts](../auth.config.ts), [proxy.ts](../proxy.ts)
- **Schema**: [prisma/schema.prisma](../prisma/schema.prisma)
- **Validations**: [lib/validations.ts](../lib/validations.ts)
- **i18n**: [lib/i18n-messages.ts](../lib/i18n-messages.ts), [lib/i18n-server.ts](../lib/i18n-server.ts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/demaceo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
