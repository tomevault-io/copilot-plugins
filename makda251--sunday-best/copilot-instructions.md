## sunday-best

> KemisHouse is a two-sided marketplace for Habesha (Ethiopian/Eritrean) traditional dresses — kemis in cotton and chiffon. Three user roles:

# KemisHouse — Habesha Dress Marketplace

## Project Overview

KemisHouse is a two-sided marketplace for Habesha (Ethiopian/Eritrean) traditional dresses — kemis in cotton and chiffon. Three user roles:

- **Buyer**: browse listings, add to cart, checkout, upload Zelle payment screenshot, track orders
- **Seller**: list dresses, manage their product catalog, fulfill orders, requires admin product approval
- **Admin**: verify Zelle payment screenshots, approve/reject seller products, manage all orders and settings

The brand tagline is "Your Home for Habesha Elegance."

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router), React 19, TypeScript |
| Styling | Tailwind CSS v4 |
| Auth + Database | Supabase (`@supabase/ssr`) |
| Email | Resend + React Email (`@react-email/components`) |
| Font | Outfit (Google Fonts, weights 300–800) |
| Images | `next/image` with Supabase Storage remote patterns |

---

## Development

```bash
npm run dev       # Start dev server on http://localhost:3000 (Turbopack)
npm run build     # Production build (Turbopack)
npm run start     # Start production server
npm run lint      # ESLint
npx tsc --noEmit  # Type-check without building
```

> **Note:** `next.config.ts` has `ignoreBuildErrors: true` and `ignoreDuringBuilds: true` for TypeScript and ESLint respectively. This allows the dev server and builds to succeed despite known pre-existing errors (see Known TypeScript Errors section below). Do not remove these flags.

---

## Project Structure

```
habesha-marketplace/
├── app/
│   ├── page.tsx                    # Home / product listing page
│   ├── layout.tsx                  # Root layout (Navbar, Outfit font)
│   ├── globals.css                 # CSS variables + Tailwind v4 @theme
│   ├── auth/
│   │   ├── login/                  # Login page
│   │   ├── signup/                 # Signup page
│   │   └── callback/               # Supabase auth callback handler
│   ├── cart/                       # Cart page
│   ├── checkout/                   # Checkout + address + payment upload
│   ├── favorites/                  # Buyer favorites page
│   ├── orders/                     # Buyer order history + detail
│   ├── products/[id]/              # Product detail page
│   ├── sellers/[id]/               # Public seller shop page
│   ├── dashboard/
│   │   ├── admin/
│   │   │   ├── page.tsx            # Admin overview
│   │   │   ├── orders/             # All orders management
│   │   │   ├── products/           # Product approval queue
│   │   │   ├── designers/          # Designer management
│   │   │   └── settings/           # Platform settings (fees, Zelle info)
│   │   ├── seller/
│   │   │   ├── page.tsx            # Seller dashboard overview
│   │   │   ├── products/           # Product CRUD (list + [id]/edit)
│   │   │   ├── orders/             # Seller order management
│   │   │   └── profile/            # Seller profile + address
│   │   └── buyer/
│   │       ├── orders/             # Buyer order history
│   │       └── profile/            # Buyer profile
│   └── api/
│       └── send-email/             # Resend email API route
├── components/
│   ├── Navbar.tsx                  # Top navigation (role-aware links)
│   ├── ProductGrid.tsx             # Product card grid for listings
│   ├── AddToCartButton.tsx         # Client component — cart interaction
│   ├── FavoriteButton.tsx          # Client component — toggle favorite
│   ├── ProductImageGallery.tsx     # Client component — image carousel
│   ├── SellerAddressPrompt.tsx     # Prompts seller to set shipping address
│   └── DashboardVisitTracker.tsx   # Tracks dashboard visits (analytics)
├── lib/
│   ├── supabase/
│   │   ├── server.ts               # createClient() for Server Components / Route Handlers
│   │   └── client.ts               # createClient() for Client Components
│   ├── types/
│   │   └── database.ts             # TypeScript types: Profile, Product, Order, etc.
│   └── email.ts                    # Resend email helpers
├── emails/                         # React Email templates
├── supabase/
│   ├── schema.sql                  # Full schema — run on fresh Supabase project
│   ├── migrations/                 # Incremental SQL migrations
│   └── *.sql                       # Standalone migration files
└── .env.example                    # Environment variable template
```

---

## Design System (CRITICAL — follow exactly)

### Color Tokens

Defined in `app/globals.css` and available via Tailwind's `@theme`:

| Token | Hex | Usage |
|---|---|---|
| `--brand` | `#C4622D` | Terracotta — buttons, logo, prices, active nav states |
| `--brand-dark` | `#A84F22` | Hover state for brand elements |
| `--brand-light` | `#FDF0EA` | Light tint backgrounds, selected states |
| `--text` | `#111111` | Primary text |
| `--text-muted` | `#6B6B6B` | Secondary / supporting text |
| `--text-subtle` | `#9A9A9A` | Placeholder, hints, disabled |
| `--bg` | `#FFFFFF` | Page backgrounds |
| `--bg-soft` | `#F7F7F7` | Input backgrounds, subtle sections |
| `--border` | `#EBEBEB` | Default borders on cards and inputs |
| `--border-strong` | `#D4D4D4` | Ghost button borders, stronger dividers |

Shadows (inline style only):
- Card: `0 1px 3px rgba(0,0,0,0.06), 0 4px 16px rgba(0,0,0,0.06)`
- Card hover: `0 4px 12px rgba(0,0,0,0.1), 0 12px 32px rgba(0,0,0,0.08)`

### Typography

**Outfit only.** Weights 300–800. Loaded via `next/font/google` in `app/layout.tsx`.

Never introduce Playfair Display, DM Sans, or any serif font.

### Tailwind v4 Color Syntax

Tailwind v4 uses bracket syntax for custom colors. When using brand colors in className strings, write:

```
bg-[#C4622D]   text-[#C4622D]   border-[#EBEBEB]   bg-[#F7F7F7]
```

However, for brand colors, **prefer inline `style` props** over Tailwind classes (see rules below).

### UI Component Patterns

Copy these patterns exactly when building new UI:

**Primary Button**
```tsx
<button
  className="rounded-full text-white font-semibold px-6 py-2.5 hover:opacity-90 transition-opacity"
  style={{ backgroundColor: '#C4622D' }}
>
  Label
</button>
```

**Ghost Button**
```tsx
<button
  className="rounded-full font-semibold px-6 py-2.5 hover:bg-[#F7F7F7] transition-colors"
  style={{ border: '1.5px solid #D4D4D4', color: '#6B6B6B' }}
>
  Label
</button>
```

**Card**
```tsx
<div
  className="bg-white rounded-2xl"
  style={{ border: '1px solid #EBEBEB' }}
>
```

**Input**
```tsx
<input
  className="rounded-lg w-full px-4 py-2 outline-none text-sm"
  style={{ border: '1px solid #EBEBEB', background: '#F7F7F7' }}
/>
```

**Terracotta Link / Text**
```tsx
<span style={{ color: '#C4622D' }}>View details</span>
// NOT: className="text-orange-600" or className="text-[#C4622D]"
```

**Status Badge (inline style helper)**
```tsx
function statusStyle(status: string): React.CSSProperties {
  switch (status) {
    case 'pending_payment': return { background: '#FEF9C3', color: '#854D0E' }
    case 'payment_verified': return { background: '#DCFCE7', color: '#166534' }
    case 'processing':       return { background: '#DBEAFE', color: '#1E40AF' }
    case 'shipped':          return { background: '#F3E8FF', color: '#6B21A8' }
    case 'delivered':        return { background: '#DCFCE7', color: '#166534' }
    case 'cancelled':        return { background: '#FEE2E2', color: '#991B1B' }
    default:                 return { background: '#F3F4F6', color: '#374151' }
  }
}

// Usage:
<span
  className="rounded-full text-xs font-medium px-2.5 py-1"
  style={statusStyle(order.status)}
>
  {order.status}
</span>
```

---

## Critical Rules

### 1. Server Components — No Mouse Event Handlers on Links

Files without `'use client'` are React Server Components. You **cannot** attach `onMouseEnter` / `onMouseLeave` to `<Link>` or any element in a Server Component.

Use Tailwind `hover:` classes instead:

```tsx
// CORRECT in Server Component:
<Link href="/products" className="hover:bg-[#F7F7F7] hover:opacity-90 transition-colors">

// WRONG — will throw at runtime:
<Link href="/products" onMouseEnter={() => setHovered(true)}>
```

If you need hover-driven state (e.g., shadow changes that can't be done in Tailwind), extract that element into a `'use client'` component.

### 2. Status Badges — Never Use Tailwind Color Classes

Always use inline style objects for status/condition/role badges. Tailwind's purge will not reliably include dynamically composed class names like `bg-green-100` built from variables.

```tsx
// CORRECT:
style={{ background: '#DCFCE7', color: '#166534' }}

// WRONG:
className="bg-green-100 text-green-800"
```

### 3. Brand Colors — Inline Style, Not Tailwind Utility

For the terracotta brand color `#C4622D`, always use `style={{ color: '#C4622D' }}` or `style={{ backgroundColor: '#C4622D' }}`.

```tsx
// CORRECT:
<p style={{ color: '#C4622D' }}>$120.00</p>

// WRONG:
<p className="text-orange-600">$120.00</p>
<p className="text-[#C4622D]">$120.00</p>  // also avoid
```

### 4. Image Component

Always use `next/image` for product images. The Supabase storage hostname is already configured in `next.config.ts`:

```
https://ttkxjchhpfgrqiajgapo.supabase.co/storage/v1/object/public/**
```

### 5. Supabase Client — Server vs Client

- **Server Components, Route Handlers, Server Actions**: import from `@/lib/supabase/server`
- **Client Components** (`'use client'`): import from `@/lib/supabase/client`

---

## Supabase

### Database Tables

| Table | Description |
|---|---|
| `profiles` | All users — role (`buyer`, `seller`, `admin`), name, phone |
| `products` | Dress listings — price, condition, size, images[], tags[], review_status |
| `orders` | Purchase orders — status, payment info, shipping address, platform_fee |
| `order_items` | Line items per order — product_id, quantity, price_at_purchase |
| `favorites` | Buyer ↔ product many-to-many |
| `admin_settings` | Platform-wide config — Zelle name/phone, platform fee % |

### Key SQL Functions

- `reserve_product(product_id, quantity)` — bypasses RLS to atomically decrement `quantity_available`. Used at checkout.

### Schema Setup

- **Fresh Supabase project**: run `supabase/schema.sql` in the SQL editor
- **Incremental changes**: apply files in `supabase/migrations/` in order

### Storage Buckets

| Bucket | Access | Contents |
|---|---|---|
| `product-images` | Public | Seller-uploaded dress photos |
| `payment-screenshots` | Private | Buyer Zelle payment proof |

### Row Level Security

RLS is enabled. The `reserve_product` SQL function uses `SECURITY DEFINER` to bypass RLS for stock decrements. All other data access goes through the anon/authenticated Supabase client.

---

## User Roles & Auth Flow

| Role | How Created | Capabilities |
|---|---|---|
| `buyer` | Self-signup at `/auth/signup` | Browse, cart, checkout, upload payment, track orders, favorites |
| `seller` | Admin-invited via `inviteUserByEmail()` | List products (pending approval), manage own orders, set shipping address |
| `admin` | Manually set in `profiles` table | Full access — verify payments, approve products, manage all orders |

Role is stored in `profiles.role`. After login, Navbar and dashboards render conditionally based on this value.

---

## Key Business Logic

### Product Approval Flow

Every new product listing goes through admin review before being visible to buyers:

```
seller creates product → review_status: 'pending'
                       → admin approves → review_status: 'approved', is_active: true
                       → admin rejects  → review_status: 'rejected' (+ rejection_reason)
```

Only `approved` + `is_active: true` products appear in public listings.

### Order / Payment Flow

KemisHouse uses Zelle as its sole payment method (no Stripe):

```
buyer checks out
  → order created with status: 'pending_payment'
  → buyer sends Zelle payment to admin's Zelle (name + phone in admin_settings)
  → buyer uploads screenshot to order
  → admin reviews screenshot → marks payment_status: 'verified'
  → order status → 'payment_verified' → 'processing' → 'shipped' → 'delivered'
```

### Refund Policy

Refunds are only issued for:
- `item_destroyed` — item arrived destroyed
- `severe_misrepresentation` — item severely different from listing

Tracked via `orders.refund_requested`, `orders.refund_reason`, `orders.refund_status`.

### Platform Fee

Tracked on `orders.platform_fee`. The fee percentage is stored in `admin_settings` and applied at checkout time.

---

## Environment Variables

Copy `.env.example` to `.env.local` for local development. Never commit `.env.local`.

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Admin email (receives order notifications)
NEXT_PUBLIC_ADMIN_EMAIL=your-email@gmail.com

# Email (Resend)
RESEND_API_KEY=your-resend-api-key
```

For production, set these in Vercel's environment variables dashboard, pointing to the production Supabase project.

---

## MCP Servers

Configure with `claude mcp add --transport http`:

```bash
claude mcp add --transport http supabase https://mcp.supabase.com/mcp
claude mcp add --transport http vercel   https://mcp.vercel.com
```

---

## TypeScript Types

Key types in `lib/types/database.ts`:

```typescript
type UserRole    = 'buyer' | 'seller' | 'admin'
type OrderStatus = 'pending_payment' | 'payment_verified' | 'processing' | 'shipped' | 'delivered' | 'cancelled' | 'refunded'
type PaymentStatus    = 'pending' | 'verified' | 'rejected'
type ProductCondition = 'new' | 'like_new' | 'excellent' | 'good' | 'fair'
```

---

## Known Pre-existing TypeScript Errors (do NOT attempt to fix)

These errors exist in the codebase and are suppressed by `ignoreBuildErrors: true` in `next.config.ts`. They do not block the dev server or builds. Do not spend time trying to fix them.

| File | Error |
|---|---|
| `emails/*.tsx` | CSSProperties type issues in React Email templates |
| `lib/email.ts` | `estimatedDelivery` optional vs required mismatch |
| `next.config.ts` | `eslint` key not valid in `NextConfig` type |
| `app/dashboard/admin/products/page.tsx` | `Product` type missing `designer` and `rejection_reason` fields |
| `app/dashboard/buyer/orders/page.tsx` | `seller` type shape mismatch on orders |
| `app/dashboard/seller/products/[id]/edit/page.tsx` | Implicit `any` on array callbacks |

---

## Planned Features

### Seller Application Flow (`/sell`)

Intake form for prospective sellers — no login required. Sellers submit contact info + dress photos. Data stored in `seller_applications` and `seller_application_items` tables. When admin approves, `inviteUserByEmail()` is called to create their Supabase account automatically and assign the `seller` role.

---

## Email

Transactional emails are sent via Resend through the `/api/send-email` route. React Email templates live in the `emails/` directory. Helper functions for triggering emails are in `lib/email.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Makda251)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Makda251)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
