## product-theqrgenapp

> This file provides guidance for AI assistants (Claude, Cursor, etc.) working on this codebase.

# CLAUDE.md - AI Assistant Instructions

This file provides guidance for AI assistants (Claude, Cursor, etc.) working on this codebase.

## Project Overview

**TheQRGenApp** is a QR code generator application with short URL routing, user authentication, and analytics tracking.

### Tech Stack
- **Frontend**: React 19 + TypeScript
- **Framework**: Next.js 16 (App Router)
- **Styling**: Tailwind CSS + shadcn/ui (Radix primitives)
- **State Management**: TanStack React Query
- **Backend**: Supabase (auth, database, edge functions)
- **QR Generation**: qr-code-styling library
- **Testing**: Vitest + React Testing Library

## Project Structure

```
app/                      # Next.js App Router pages
├── layout.tsx            # Root layout (providers, metadata)
├── providers.tsx         # Client-side providers
├── page.tsx              # Landing page (/)
├── not-found.tsx         # 404 page
├── auth/page.tsx         # /auth
├── dashboard/page.tsx    # /dashboard
├── generate/page.tsx     # /generate
├── account/page.tsx      # /account
├── subscription/page.tsx # /subscription
├── terms/page.tsx        # /terms
├── privacy/page.tsx      # /privacy
├── cookie-settings/page.tsx
├── go/[shortCode]/page.tsx  # QR redirect handler
└── qr/[id]/edit/page.tsx    # Edit QR code

src/
├── components/           # React components
│   ├── ui/              # shadcn/ui components (auto-generated)
│   ├── AppNav.tsx       # Main navigation bar
│   ├── QRGenerator.tsx  # QR code generator for authenticated users
│   ├── QRCodeCard.tsx   # QR code display card
│   └── ...
├── hooks/               # Custom React hooks
│   ├── use-require-auth.ts  # Auth guard hook
│   ├── use-mobile.tsx       # Mobile detection
│   └── use-toast.ts         # Toast notifications
├── lib/                 # Utility functions
│   ├── qr-utils.ts      # QR code utilities, short codes, expiration
│   ├── user-agent.ts    # User agent parsing
│   └── utils.ts         # General utilities (cn, etc.)
├── types/               # TypeScript interfaces
│   └── index.ts         # Centralized type definitions
├── config/              # Configuration files
│   └── navigation.ts    # Navigation items
├── contexts/            # React contexts
│   └── SubscriptionContext.tsx
├── integrations/
│   └── supabase/        # Supabase client and generated types
├── __tests__/           # Unit tests
│   ├── setup.ts         # Test configuration
│   ├── qr-utils.test.ts # QR utilities tests
│   └── user-agent.test.ts # User agent tests
└── index.css            # Global styles

supabase/
├── functions/           # Edge functions
│   ├── track-scan/      # Analytics tracking
│   ├── check-subscription/
│   ├── create-checkout/
│   └── customer-portal/
└── migrations/          # Database migrations

docs/
└── ai/
    └── skills/          # AI assistant skills (custom instructions)
```

## Key Concepts

### QR Code System
- All QR codes go through internal routing: `/go/:shortCode`
- Short codes are 8-character alphanumeric, case-insensitive
- Guest codes expire in 7 days
- Free account codes expire in 90 days
- Pro codes never expire

### Database Tables (Supabase)
- `qr_codes`: QR code data with `short_code`, `is_guest`, `scan_count`
- `qr_analytics`: Scan tracking data
- `profiles`: User profiles
- `subscriptions`: Stripe subscription data
- `qr_folders`: QR code organization

### Button Styles
Custom depth button variants are available:
- `variant="depth"` - Standard 3D button with shadow
- `variant="depth-cta"` - Inverted CTA button (pressed state by default)
- `variant="depth-primary"` - Primary action with depth effect

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Lint code
npm run lint

# Run tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

## Environment Variables

Required in `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your-anon-key
```

## Coding Conventions

### File Organization
- Pages go in `app/` directory (Next.js App Router)
- Reusable components go in `src/components/`
- Client components need `"use client"` directive
- Hooks use kebab-case with `use-` prefix: `use-require-auth.ts`
- Utilities use kebab-case: `qr-utils.ts`

### TypeScript
- All types in `src/types/index.ts` for shared types
- Component-specific types can be inline
- Use strict typing, avoid `any`

### Styling
- Use Tailwind CSS utility classes
- Use CSS variables from `index.css` for theming
- Follow existing patterns in shadcn/ui components

### Components
- Use `"use client"` for interactive components
- Use `useRequireAuth()` hook for protected pages
- Use `NAV_ITEMS` from config for consistent navigation
- Use Next.js `Link` and `useRouter` for navigation

### Testing
- Unit tests go in `src/__tests__/`
- Use Vitest for testing
- Test utility functions thoroughly
- Mock Supabase and browser APIs as needed

## Common Tasks

### Adding a New Page
1. Create page component in `app/route-name/page.tsx`
2. Add `"use client"` if page has interactivity
3. Add to navigation in `src/config/navigation.ts` if needed

### Adding a New Component
1. Create in `src/components/`
2. Add `"use client"` if it uses hooks/state
3. Export and import where needed

### Database Changes
1. Create migration in `supabase/migrations/`
2. Update types in `src/integrations/supabase/types.ts`
3. Update `src/types/index.ts` if shared types change

## Gotchas

- QR code rendering requires a DOM ref and `useEffect` - watch for race conditions
- Guest QR generation has a "local mode" fallback when Supabase is unavailable
- The landing page has both authenticated and guest states
- shadcn/ui components are in `src/components/ui/` - don't modify directly
- All pages with interactivity need `"use client"` directive

## Skills Directory

Custom skills/instructions are in `docs/ai/skills/`. Each skill should be a markdown file with specific instructions for particular tasks or domains. See `docs/ai/skills/README.md` for details on adding new skills.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/godsone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
