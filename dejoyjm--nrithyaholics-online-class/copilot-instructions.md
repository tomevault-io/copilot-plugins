## nrithyaholics-online-class

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run frontend dev server (from /frontend)
cd frontend && npm run dev        # http://localhost:5173

# Lint
cd frontend && npm run lint

# Build for production
cd frontend && npm run build

# Deploy edge functions (requires Supabase CLI linked to project)
supabase functions deploy <function-name>
# e.g. supabase functions deploy send-join-links

# Deploy all functions
supabase functions deploy

# Set secrets (one-time setup)
supabase secrets set KEY=value
```

There are no automated tests. No test runner is configured.

## Architecture

This is a **single-page React app** with **Supabase as the entire backend** (DB + auth + edge functions). There is no Express/Node server.

### Navigation model

`App.jsx` is a **single-component router** — it holds all top-level state and renders one "page" at a time by conditionally returning a page component. There is no React Router or URL-based routing. Page transitions are driven by state variables (`currentSession`, `currentClassroom`, `showProfile`, `mode`, `currentChoreoId`, etc.).

Priority order of what renders (top to bottom in `App.jsx`):
1. Loading spinner
2. `SuspendedPage` (if `profile.suspended`)
3. `AuthPage` (if `showAuth && !user`)
4. `AdminPage` (if `profile.is_admin`) — admin bypasses all other flows
5. `RoleSelectPage` (if user has no role yet)
6. `ClassroomPage` (if `currentClassroom` is set)
7. `ChoreoPage` (if choreographer in teaching mode)
8. `ProfilePage`, `ChoreoProfilePage`, `SessionPage` (by respective flags)
9. `HomePage` (default)

### Database (Supabase/Postgres)

Key tables:
- `profiles` — extends `auth.users`; fields: `role` (learner/choreographer), `is_admin`, `choreographer_approved`, `suspended`
- `profiles_with_email` — view joining `profiles` + `auth.users.email` (admin use only)
- `sessions` — dance classes; fields: `choreographer_id`, `scheduled_at`, `duration_mins`, `max_seats`, `status` (draft/open/confirmed/cancelled/completed), `price_tiers` (JSON array)
- `bookings` — links users to sessions; `seats`, `amount_paid_inr`, `razorpay_order_id`
- `platform_config` — single row (id=1); controls pre-join/grace time windows for host and guest

RLS is enabled. Admin operations that bypass RLS use `profiles_with_email` view or service role key in edge functions.

### Edge Functions (`supabase/functions/`)

All written in Deno/TypeScript. Called via `fetch` directly from frontend using `VITE_SUPABASE_URL/functions/v1/<name>`.

| Function | Purpose |
|---|---|
| `create-razorpay-order` | Creates Razorpay order before payment |
| `verify-payment` | Frontend fallback: verifies signature + creates booking |
| `razorpay-webhook` | Primary path: Razorpay calls this on payment success; creates booking + sends confirmation email via Resend |
| `razorpay-callback` | Redirect URL after Razorpay payment (used for mobile/redirect flow) |
| `get-token` | Issues 100ms room JWT for ClassroomPage; enforces time-gate (too early / grace period) |
| `send-join-links` | Cron job — sends join link emails 5 min before each session via Resend |

### Payment flow (dual-path)

1. **Webhook path (primary):** Razorpay → `razorpay-webhook` edge fn → creates booking → redirects to `?payment_success=1&session_id=X`
2. **Frontend path (fallback):** Razorpay SDK calls `handler` callback → App stores params in `sessionStorage` → redirects → `verify-payment` edge fn creates booking

Both paths converge in `SessionPage.jsx` via the `razorpayReturn` prop passed from `App.jsx`.

### Live classroom (100ms)

`ClassroomPage.jsx` calls `get-token` edge fn to get a JWT for the 100ms room, then renders a `<iframe>` pointing to `https://nrithyaholics.app.100ms.live/...`. The `get-token` function enforces:
- Host can join `host_pre_join_minutes` before scheduled time
- Guest can join `guest_pre_join_minutes` before scheduled time
- Grace period extends join window after scheduled end

Ghost peer detection is server-side: `get-token` calls the 100ms `/active-rooms` API to check for stale peers rather than trusting client-reported state.

### Email

All transactional email goes through **Resend** (`bookings@nrithyaholics.in`). Emails sent:
- Booking confirmation (from `razorpay-webhook`)
- Join link 5 min before class (from `send-join-links` cron)

Deep links in emails use `?session=ID` and `?session=ID&test=1` query params. `App.jsx` parses these on load and routes accordingly. `cameFromEmail` flag is passed to `SessionPage` to show a contextual login CTA.

### Environment variables

Frontend (in `frontend/.env.local`):
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

Edge functions (set via `supabase secrets set`):
- `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `RAZORPAY_WEBHOOK_SECRET`
- `RESEND_API_KEY`, `RESEND_FROM_EMAIL`
- `HMS_ACCESS_KEY`, `HMS_APP_SECRET` (100ms credentials)

### Deployment

- Frontend: Vercel (`https://nrithyaholics-online-class.vercel.app` / custom domain `https://online.nrithyaholics.in`)
- Backend: Supabase project `nrithyaholics-online-class` (ref: `vuxqimoqsbqsgvkashak`)
- Edge functions deployed via Supabase CLI

---
> Source: [dejoyjm/nrithyaholics-online-class](https://github.com/dejoyjm/nrithyaholics-online-class) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
