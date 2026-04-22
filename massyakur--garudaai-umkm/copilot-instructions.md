## garudaai-umkm

> This is the working brief for the NEVADA (Nusantara Enterprise Virtual Assistant & Data Analytics) frontend. Use it as a source of truth when making changes.

# CLAUDE.md - NEVADA Frontend Context

This is the working brief for the NEVADA (Nusantara Enterprise Virtual Assistant & Data Analytics) frontend. Use it as a source of truth when making changes.

## Project Overview
- **Framework**: Next.js 15 (App Router) + TypeScript
- **UI**: Tailwind CSS v4 + shadcn/ui + Lucide icons
- **Auth**: Custom JWT from FastAPI (`POST /api/v1/login`), stored via `AuthContext` (localStorage)
- **Backend**: Custom FastAPI base URL from `NEXT_PUBLIC_API_URL`
- **State**: Client components for forms/tables; lightweight context for auth only
- **Design**: Warm amber/emerald gradient identity, mobile-friendly layouts

## Architecture
```
src/
  app/
    (app)/layout.tsx          # Protected shell + nav + auth guard
    (app)/dashboard/page.tsx  # Analytics dashboard
    (app)/products/page.tsx   # Product CRUD
    (app)/customers/page.tsx  # Customer CRUD + search
    (app)/transactions/page.tsx # Transactions CRUD + filters + OCR upload
    (app)/reports/page.tsx    # Sales/monthly/top-products reports
    (app)/content-agent/page.tsx # FastAPI content agent chat UI
    login/page.tsx            # JWT login
    page.tsx                  # Landing/marketing
  components/
    ui/*                      # shadcn primitives
    theme-provider.tsx, theme-toggle.tsx
  context/AuthContext.tsx     # JWT + user, login/logout helpers
  lib/api.ts                  # All FastAPI calls + error handling
  lib/utils.ts                # `cn` helper
```

## API Layer (`src/lib/api.ts`)
- Base URL: `NEXT_PUBLIC_API_URL` (trimmed)
- `ApiError` with status + details for surfaced errors
- Helpers include:
  - `login(email, password)` → `{ access_token, user }`
  - Products: `getProducts`, `createProduct`, `updateProduct`, `deleteProduct`
  - Customers: `getCustomers`, `createCustomer`, `updateCustomer`, `deleteCustomer`
  - Transactions: `getTransactions`, `createTransaction(token, payload, userId?)`, `updateTransaction`, `deleteTransaction`
  - Analytics: `getDashboard`, `getSalesReport`, `getTopProducts`, `getMonthlyReport`, `getPaymentMethods`
  - OCR: `uploadReceipt(token, file, extra?)` posts to `/api/v1/ocr/upload`
  - Content agent: `sendContentMessage`, `getContentHistory`, `clearContentHistory`, `deleteContentThread`
  - `healthCheck` for root endpoint
- Query strings built via `buildQueryString`; FormData respected (no forced content-type)

### Data Models (lightweight)
- `User`: `{ id, email?, name?, umkm_id?, role? }`
- `Product`, `Customer`, `Transaction` typed with optional fields to match FastAPI schemas; handle `umkm_id` consistently.

## Auth (`src/context/AuthContext.tsx`)
- Stores `token` + `user` in localStorage keys `nevada_token` / `nevada_user`
- Exposes `{ user, token, loading, isAuthenticated, login, logout }`
- `login` calls API helper, sets state + storage; `logout` clears both
- Used by `(app)/layout.tsx` guard + header/logout controls

## Protected Shell (`src/app/(app)/layout.tsx`)
- Client layout that redirects to `/login` when unauthenticated
- Sidebar + top bar navigation for Dashboard, Products, Customers, Transactions, Reports, Content Agent
- Mobile nav via `Sheet`, theme toggle, logout button

## Pages
- **Landing (`page.tsx`)**: Marketing hero for NEVADA with CTA to login, highlights OCR/analytics/content agent.
- **Login**: Email/password form hitting `/api/v1/login`; redirects to dashboard on success.
- **Dashboard**: Cards for revenue/transactions/customers/products, daily sales chart (Recharts via shadcn `ChartContainer`), payment mix, top products, credit-readiness note. Refresh accepts `umkm_id` + `days`.
- **Products**: Table + dialog for create/edit, delete confirmation, filters (category/status), uses `umkm_id`.
- **Customers**: Table + dialog for create/edit, search input, delete confirmation.
- **Transactions**: CRUD with status/date filters, product/customer IDs, notes. OCR modal uploads receipt to `/api/v1/ocr/upload`, shows extracted text, can inject into transaction form. `user_id` passed to create when available.
- **Reports**: Calls `/analytics/sales-report`, `/analytics/monthly-report`, `/analytics/top-products`; configurable date range and months.
- **Content Agent**: Chat UI sending text + optional image to `/content-agent/chat`, shows history, clear/delete thread actions.

## Environment Variables
```
NEXT_PUBLIC_API_URL=https://9812afe22a53.ngrok-free.app
```

## Removed/Deprecated
- All Clerk and Supabase integrations, middleware, and env keys have been removed in favor of direct FastAPI calls.
- Legacy Supabase migrations and Clerk setup docs are no longer used; keep ignored unless you intentionally reintroduce them.

## UI/UX Notes
- Palette: amber/emerald gradients, glassmorphism cards; avoid purple defaults.
- Typography: Geist + Parkinsans; keep copy concise and in English (UI may include Indonesian phrases where meaningful).
- Mobile: nav collapses into `Sheet`, forms/tables are scrollable.
- Comments: only where logic is non-obvious (avoid noise).

## Workflows to Test
1) Login → redirect to dashboard (JWT stored)
2) Create/edit/delete product (UMKM ID attached) and refresh list
3) Create/edit/delete customer; search filter works
4) Create transaction; filter by status/date; delete; ensure amount displayed
5) OCR upload → text returned → pushed into transaction form
6) Dashboard refresh with `umkm_id` + `days`
7) Reports refresh with date range + months
8) Content agent chat, reload history, clear/delete thread

## Conventions
- Use `api.ts` helpers for all backend calls; handle errors with `toast` where user-facing.
- Keep everything in English in code/comments; UI may mix Indonesian terms for clarity.
- Avoid reintroducing external auth/db services without aligning with the FastAPI contract.
- Default to ASCII text. Remove legacy Clerk/Supabase mentions if encountered in code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/massyakur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
