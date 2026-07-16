## certipure-store

> This file gives Claude Code context about this project. Claude Code reads it

# CLAUDE.md — CertiPure Store

This file gives Claude Code context about this project. Claude Code reads it
automatically at the start of every session.

## About the person you're working with

- Josh is **not a coder**. Always explain things in plain English.
- Do not assume background knowledge about web development, terminals, or code.
- When suggesting a change, explain *what* it does and *why* before doing it.
- Give **step-by-step instructions** — small, numbered steps, one action at a time.
- After making changes, briefly state what changed and what (if anything) Josh
  should check or test in the browser.
- If something could break the site, say so clearly before doing it.

## What this project is

CertiPure is a **research peptides e-commerce store**. It's a website where
customers can browse products, create an account, and (eventually) check out
and pay.

The site has a scientific blue-and-white look with DNA helix imagery and the
tagline "Tested – Trusted – Affordable."

## Tech stack

- **Next.js 16** — the website framework (App Router; `src/` structure).
  Note: page `params` arrive as a Promise and must be `await`ed.
- **Supabase** — handles user accounts (signup/login), the database, and file
  storage (payment screenshots).
- **Resend** — sends transactional emails (order receipts, admin alerts).
- **Crypto checkout** — payment is taken via cryptocurrency with manual
  verification (see "Checkout & orders" below). This replaced the original
  Stripe plan. Stripe keys still exist in `.env.local` but are unused/shelved.
- **Vercel** — where the site will be deployed/hosted (planned).
- Code is developed through **Claude Code**, not by hand-editing files.

## Project setup

- GitHub repo: github.com/joshuariley76/certipure-store
- Local project folder: OneDrive/Desktop/certipure-store
- The project uses a **`src/` folder structure**.

## Important rules and cautions

- **Do not modify `src/lib/supabase.ts`** unless it is directly causing a
  breakage. Four pages depend on this file, so changes there can break the
  whole site.
- Before deleting or replacing anything, confirm with Josh first.
- Prefer small, reversible changes over large rewrites.

## What's already done

- Authentication system is complete:
  - Supabase signup/login gate
  - GateModal with a blurred homepage preview
  - Email confirmation
  - Profile creation via a database trigger
  - Sign Out button in the Navbar
  - Terms and Privacy pages are live
- Shopping cart (Supabase-backed, per user) with slide-in drawer and `/cart`.
- **Crypto checkout + order processing** (see next section).
- **Admin area** to review orders and verify payments (see next section).

## Checkout & orders (built)

**Customer flow:** Cart → `/checkout` → fills shipping form, picks a coin, sees
the matching wallet address, uploads a payment screenshot → on submit the order
is saved and they land on `/order-confirmed/<orderNumber>`.

- **Checkout form:** `src/components/CheckoutClient.tsx` (rendered by
  `src/app/checkout/page.tsx`). Wallet addresses come from `NEXT_PUBLIC_WALLET_*`
  env vars.
- **Order API:** `src/app/api/create-order/route.ts` — verifies the signed-in
  user, validates the screenshot (image, ≤10 MB), uploads it to the
  `order-screenshots` Supabase storage bucket, inserts rows into the `orders`
  and `order_items` tables, clears the cart, then emails the customer a receipt
  and the admin a notification via Resend. Generates an order number like
  `CP-XXXXX`.
- **Confirmation page:** `src/app/order-confirmed/[orderNumber]/page.tsx`.
- **Email client:** `src/lib/resend.ts`. Emails send from `@certipure.net`
  (domain verified in Resend); admin alerts go to `joshua@certipure.net`.

**Admin flow:** `/admin/login` (password) → `/admin/orders` (review every order,
view the payment screenshot, mark Verified / Shipped / Cancelled).

- Login + password logic: `src/lib/admin-auth.ts` (password = `ADMIN_KEY` env
  var; login sets an httpOnly cookie — the password is never stored client-side).
- Pages: `src/app/admin/login/page.tsx`, `src/app/admin/orders/page.tsx`,
  `src/app/admin/orders/OrdersAdmin.tsx`.
- APIs: `src/app/api/admin/{login,logout,update-order}/route.ts`.
- The admin pages read ALL orders using a service-role Supabase client
  (`src/lib/supabase/admin.ts`), which needs `SUPABASE_SERVICE_ROLE_KEY`. This
  client is server-only and must never be imported into browser code.

**Required env vars (`.env.local`, not committed):**
`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`,
`SUPABASE_SERVICE_ROLE_KEY` (secret), `RESEND_API_KEY` (secret),
`NEXT_PUBLIC_WALLET_BTC/ETH/USDT/USDC/SOL`, `ADMIN_KEY` (secret).
After editing `.env.local`, the dev server must be restarted to pick up changes.

**Supabase pieces this depends on (already set up):** tables `orders` and
`order_items`, and a storage bucket `order-screenshots`.

## Current priorities (in order)

1. **Fix landing page issues** — duplicate/placeholder products, garbled footer
   text, the value proposition section, design contrast, and social proof.
2. **Vercel deployment** — get the site live, including updating Supabase URL
   and redirect settings, and adding all `.env.local` variables to Vercel.
3. **Email confirmation landing page** — a proper page after a user confirms
   their email.
4. **Customer order history page** — where logged-in users can see their own
   past orders (the admin order view already exists; this is the customer side).

## Security reminders

- `ADMIN_KEY` should be a strong, private password (the original value was a
  weak placeholder from old code). Change it before going live.
- Never share or expose `SUPABASE_SERVICE_ROLE_KEY` or `RESEND_API_KEY`; keep
  them out of any committed file (`.env.local` is gitignored).

## Style of help Josh wants

- Very specific, detailed instructions.
- Plain English, no unexplained jargon.
- Help with both updating the site and optimizing it.

---
> Source: [joshuariley76/certipure-store](https://github.com/joshuariley76/certipure-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
