## pawples

> Pawples (thepawples.com) is an **Indian pet e-commerce platform** with a core subscription box product. The mission: deliver curated, vet-approved treats, toys, and wellness products for Indian pets every month. Tagline: **"Because they're family."**

# Pawples — Project Context for Claude

## What is Pawples?
Pawples (thepawples.com) is an **Indian pet e-commerce platform** with a core subscription box product. The mission: deliver curated, vet-approved treats, toys, and wellness products for Indian pets every month. Tagline: **"Because they're family."**

Target market: India (INR pricing, Razorpay payments, pan-India shipping via Shiprocket/Delhivery).
Pet categories: **Dogs, Cats, Birds, Small Animals** (hamsters, rabbits, guinea pigs, etc.).

---

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 14 — App Router |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS (custom brand config) |
| Database & Auth | Supabase (Postgres + Supabase Auth) |
| Payments | Razorpay (orders + subscriptions) |
| Email | Resend |
| Image CDN | Cloudinary |
| Fonts | Nunito (headings, 800/900 weight) + DM Sans (body) from Google Fonts |

---

## Brand

```
Primary (Plum):    #5B2D8E
Accent (Gold):     #F5A623
Background (Ivory):#FDF8F2
Text (Deep Bark):  #1A1A2E
```

**Logo mark**: A plum-coloured ellipse (nose shape) with two gold nostril ellipses inside — rendered as an inline SVG in `components/layout/PawplesLogo.tsx`.

Tailwind colour tokens: `plum`, `gold`, `ivory`, `bark` (see `tailwind.config.ts`).
Font family tokens: `font-nunito`, `font-dm-sans`.

---

## Project Structure

```
/
├── app/                        # Next.js App Router pages
│   ├── layout.tsx              # Root layout (fonts, metadata)
│   ├── globals.css             # Tailwind base + global styles
│   ├── page.tsx                # Homepage (fully built)
│   ├── shop/
│   │   ├── page.tsx            # All-products shop
│   │   ├── [category]/         # Category page (dogs/cats/birds/small-animals)
│   │   └── [category]/[slug]/  # Product detail page (PDP)
│   ├── subscribe/
│   │   ├── page.tsx            # Subscription landing
│   │   ├── quiz/               # Pet personalisation quiz
│   │   ├── choose/             # Plan selector (Snoozy/Zoomies/Royale)
│   │   ├── addons/             # Addon picker
│   │   └── success/            # Post-subscription confirmation
│   ├── search/                 # Search results
│   ├── about/ blog/ contact/   # Informational pages
│   ├── login/ signup/          # Auth pages
│   ├── forgot-password/        # Password reset
│   ├── onboarding/             # Post-signup pet setup
│   ├── cart/                   # Cart page
│   ├── checkout/               # Checkout (Razorpay integration)
│   ├── order/[id]/             # Order detail / tracking
│   ├── account/                # Account dashboard
│   │   ├── orders/             # Order history
│   │   ├── subscription/       # Manage subscription
│   │   ├── pets/               # Pet profiles
│   │   ├── addresses/          # Saved addresses
│   │   ├── rewards/            # Reward points / referrals
│   │   └── settings/           # Profile & notification settings
│   ├── admin/                  # Internal admin (protected)
│   │   ├── products/           # Product management
│   │   ├── orders/             # Order management
│   │   ├── subscriptions/      # Subscription management
│   │   ├── customers/          # Customer management
│   │   └── promos/             # Promo code management
│   └── privacy/ terms/ shipping/ returns/  # Legal & policy pages
│
├── components/
│   ├── layout/
│   │   ├── PawplesLogo.tsx     # SVG logo mark + wordmark
│   │   ├── Navbar.tsx          # Sticky nav with mobile drawer
│   │   └── Footer.tsx          # Full footer with social links
│   ├── ui/
│   │   ├── Button.tsx          # btn-primary / secondary / outline / ghost
│   │   └── Badge.tsx           # Colour-coded badges
│   ├── shop/                   # Product cards, filters, gallery
│   ├── subscription/           # Plan cards, quiz steps, addon picker
│   ├── account/                # Account section components
│   └── admin/                  # Admin table, stat cards, etc.
│
├── lib/
│   ├── supabase.ts             # Supabase client (anon) + supabaseAdmin()
│   ├── razorpay.ts             # Razorpay client, createOrder, verifySignature
│   └── utils.ts                # cn(), formatPrice(), slugify(), etc.
│
├── types/
│   └── index.ts                # All TS types: Product, Order, Subscription, Pet, User, CartItem, etc.
│
├── public/
│   └── images/                 # Static assets
│
├── CLAUDE.md                   # ← This file
├── .env.local.example          # Required env vars (copy → .env.local)
├── tailwind.config.ts          # Brand colours + fonts + animations
├── next.config.ts              # Image domains (picsum, cloudinary)
└── tsconfig.json               # Strict TS, path alias @/*
```

---

## Key Types (types/index.ts)

- **Product** — slug, variants (SKU/price/stock), category, rating, status
- **ProductVariant** — id, sku, price (paise), stock, weight, attributes
- **CartItem** — productId, variantId, name, image, price, quantity
- **Order** / **OrderItem** — status lifecycle: pending → confirmed → processing → shipped → out_for_delivery → delivered
- **Subscription** — plan (snoozy/zoomies/royale), status, frequency, Razorpay subscription ID
- **Pet** — type (dog/cat/bird/small-animal), breed, age, allergies, preferences
- **User** — role (customer/admin/moderator), rewardPoints, referralCode
- **PromoCode** — discountType (percentage/fixed/free_shipping)

All monetary values are stored in **paise** (₹1 = 100 paise). Use `formatPrice(paise)` from `lib/utils.ts` for display.

---

## Subscription Plans

| Plan | Price | Items | Key Feature |
|---|---|---|---|
| Snoozy | ₹599/mo | 5 | Budget-friendly, chill pets |
| Zoomies | ₹999/mo | 8 | Most popular, active pets |
| Royale | ₹1,799/mo | 12 | Luxury, includes personalised card |

All plans have monthly frequency by default. Razorpay subscription API (`razorpay.subscriptions.create`) powers recurring billing.

---

## Payments (Razorpay)

- One-time orders: `razorpay.orders.create()` → verify with HMAC-SHA256 signature
- Subscriptions: `razorpay.subscriptions.create()` with plan and customer details
- Helper in `lib/razorpay.ts`: `createRazorpayOrder()`, `verifyRazorpaySignature()`
- Client-side: load Razorpay JS SDK, call `new window.Razorpay(options).open()`
- All amounts in paise; currency = "INR"

---

## Auth (Supabase)

- Auth provider: Supabase Auth (email/password + Google OAuth)
- Server components: use `createServerClient` from `@supabase/ssr`
- Client components: use the singleton from `lib/supabase.ts`
- `supabaseAdmin()` (service role) is server-only — never import in client components
- Protected routes: middleware checks session; redirect to /login if not authenticated
- Admin routes (`/admin/*`): check `user.role === 'admin'`

---

## Styling Conventions

- Use `cn()` from `lib/utils.ts` (clsx + tailwind-merge) for conditional classes
- Buttons: use `components/ui/Button.tsx` — variants: primary, secondary, outline, ghost
- Cards: use `.card` utility class (defined in globals.css)
- Headings always use `font-nunito font-extrabold` (or `font-black` for hero)
- Body text: `font-dm-sans`
- Section spacing: `py-20` on sections, `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8` for containers
- Rounded corners: `rounded-3xl` for cards, `rounded-full` for buttons/pills, `rounded-5xl` for hero/banner blocks

---

## India-Specific Notes

- Currency: always display as `₹` with Indian number formatting (`en-IN` locale)
- Phone: validate with `/^[6-9]\d{9}$/` (10-digit, starts 6-9)
- Pincode: 6-digit, validate with `/^[1-9][0-9]{5}$/`
- GST: 18% on most pet products (some exemptions for food)
- Shipping partner: Shiprocket (primary), Delhivery (backup)
- WhatsApp notifications for order updates (India preferred channel)

---

## Dev Commands

```bash
npm run dev      # Start dev server at localhost:3000
npm run build    # Production build
npm run lint     # ESLint check
```

Copy `.env.local.example` → `.env.local` and fill in values before running.

---

## Placeholder Images

During development, images use `picsum.photos` with seeds for consistency:
- `https://picsum.photos/seed/{seedname}/{width}/{height}`
- Seeds used: `pawdog`, `pawcat`, `pawbird`, `pawsmall`, `pawpleshero`, `prod1paw`–`prod4paw`, `box1`–`box4`, `review1`–`review3`

Replace with Cloudinary URLs before production.

---
> Source: [shivaprasad909/Pawples](https://github.com/shivaprasad909/Pawples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
