## dealkart

> DealKart is India's first verification-first marketplace for buying/selling MSME businesses.

# DealKart — MSME M&A Marketplace MVP

## What Is This?
DealKart is India's first verification-first marketplace for buying/selling MSME businesses.
Think "Zillow for businesses" — CA-verified financials, NDAs, structured deal flow.
Target: 63M+ Indian MSMEs. Deal range: ₹50L — ₹100Cr.

## Tech Stack
- **Next.js 14** (App Router, TypeScript, `src/` directory)
- **Supabase** (PostgreSQL + Auth + Storage + Row Level Security)
- **Tailwind CSS + shadcn/ui** for UI components
- **React Hook Form + Zod** for form validation
- **Zustand** for client-side state
- **sonner** for toast notifications
- **lucide-react** for icons

## Project Structure
```
src/
├── app/
│   ├── auth/             # Login, Register (public routes)
│   ├── seller/           # Seller routes (role=SELLER)
│   │   ├── onboarding/   # 4-layer wizard
│   │   └── dashboard/
│   ├── buyer/            # Buyer routes (role=BUYER)
│   │   ├── marketplace/
│   │   └── listing/[id]/
│   ├── admin/            # Admin routes (role=ADMIN)
│   │   ├── dashboard/
│   │   ├── listings/
│   │   ├── users/
│   │   └── ndas/
│   ├── api/
│   ├── page.tsx          # Landing page
│   └── layout.tsx
├── components/
│   ├── ui/               # shadcn (DO NOT EDIT manually)
│   ├── auth/
│   ├── seller/
│   ├── buyer/
│   ├── admin/
│   └── shared/
├── lib/
│   ├── supabase/         # client.ts, server.ts, middleware.ts
│   ├── validations/      # Zod schemas
│   ├── constants.ts
│   └── utils.ts
├── hooks/
├── types/database.ts
└── stores/
```

## Database Tables
- `profiles` — extends auth.users, has role (SELLER/BUYER/ADMIN)
- `seller_profiles` — business identity + onboarding state
- `seller_financials` — revenue, EBITDA, asking price (stored in paise as BIGINT)
- `seller_documents` — uploaded file metadata
- `seller_preferences` — deal preferences, T&C acceptance
- `listings` — marketplace listings with status workflow
- `ndas` — buyer NDA records
- `saved_listings` — buyer watchlist

## CRITICAL Coding Rules
1. **Server Components by default.** Only add `"use client"` for forms, hooks, interactivity.
2. **All DB queries through Supabase client** — never raw SQL in components.
3. **Zod schemas** for ALL form validation (shared client + server).
4. **TypeScript strict mode.** NEVER use `any`. Define proper types.
5. **Financial values**: store as BIGINT (paise) in DB, format to ₹ in UI.
6. **Indian formatting**: `new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR', maximumFractionDigits: 0 })` → ₹12,34,567
7. **Error handling**: try/catch with `toast.error()` from sonner.
8. **lucide-react** for all icons. No other icon libraries.
9. **shadcn/ui** for all UI components. Import from `@/components/ui/`.
10. **Route protection**: middleware checks `profiles.role` for seller/buyer/admin routes.

## Design System
- **Primary**: emerald-600 (#059669) — trust, growth
- **Secondary**: slate-800 (#1e293b) — professional
- **Accent**: amber-500 (#f59e0b) — warnings, attention
- **Success**: green-500, **Error**: red-500
- **Border radius**: rounded-lg (default), rounded-xl (cards)
- **Shadows**: shadow-sm (default), shadow-lg (hover/elevated)
- **Font**: system font stack. No external fonts.

## Auth Flow
- Supabase Auth with email magic link (MVP — phone OTP is Phase 2)
- Role passed in `user_metadata` during signup → trigger creates profile row
- Middleware reads `profiles.role` and protects routes accordingly
- Login: email → magic link → redirect by role
- Register: choose role → enter details → magic link → redirect

## User Roles & Permissions
| Action | SELLER | BUYER | ADMIN |
|--------|--------|-------|-------|
| Onboard | ✓ | ✗ | ✗ |
| Create listings | ✓ | ✗ | ✗ |
| Browse marketplace | ✓ | ✓ | ✓ |
| Sign NDA | ✗ | ✓ | ✗ |
| Save listings | ✗ | ✓ | ✗ |
| Approve listings/NDAs | ✗ | ✗ | ✓ |
| Manage users | ✗ | ✗ | ✓ |

## Business Rules
- Min revenue: ₹25L. Business age: 2+ years.
- Max 5 active NDAs per buyer.
- Seller identity NEVER shown on marketplace.
- Listing statuses: DRAFT → PENDING_REVIEW → LIVE → UNDER_OFFER → CLOSED

## When Creating Components
- Always co-locate related components (e.g., seller components in `components/seller/`)
- Use descriptive file names: `ListingCard.tsx`, `OnboardingStepper.tsx`
- Keep components focused — one component per file
- Extract reusable logic into hooks (`hooks/`)
- Use `cn()` utility from `lib/utils.ts` for conditional classNames

---
> Source: [cksynapse-oss/dealkart](https://github.com/cksynapse-oss/dealkart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
