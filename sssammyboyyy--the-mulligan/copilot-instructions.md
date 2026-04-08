## the-mulligan

> "Industrial Security Refactor: 2026-03-17 18:13 SAST"

# 🏌️ GEMINI.md — The Mulligan (Venue OS)
"Industrial Security Refactor: 2026-03-17 18:13 SAST"

---

This document provides the "Genetic Code" for The Mulligan's booking engine. Use this as a RAG brain for all future tasks.

## 🏛️ Core Architecture: "The Direct Highway"
- **Edge Native**: The application uses the OpenNext Cloudflare adapter. Do NOT use `export const runtime = 'edge'` in individual routes; it MUST be standard Node runtime to prevent bundling conflicts.
- **Self-Healing Engine**: Do not trust local state for payments. The reconciliation worker verifies pending checkouts against Yoco's API.
- **Ghost Cleanup Protocol**: Hard-delete logic is implemented in `/api/bookings/admin-delete` to immediately release PostgreSQL `EXCLUDE USING gist` constraints.
- **Atomic Guards**: Use `booking_request_id` (Unique) for creation and `email_sent` (Atomic Bool) for automation triggers.

## ⚖️ POS Pricing Engine (Tiered Rates)
- **1 Player**: R250/hr
- **2 Players**: R360/hr
- **3 Players**: R480/hr
- **4 Players**: R600/hr
- **Add-ons**: 
  - Clubs (R100/hr)
  - Coaching (R250 Flat)
  - Retail: Support for manager price persistence via `addon_*_price` columns.

## 🛠️ Tech Stack & Rules
1. **Frontend**: Next.js 15 (App Router).
2. **Styling**: Vanilla CSS only. No Tailwind. (Verified: `tailwindcss` in `package.json` is currently unused).
3. **Database**: Supabase PostgreSQL.
   - **Double-Booking Prevention**: Handled via `EXCLUDE USING gist` constraint on `(simulator_id, tstzrange(slot_start, slot_end))`.
   - **View Layer**: Always use `booking_dashboard` view for admin UIs to ensure consistent payment state calculation.
4. **Automation**: n8n orchestration.
   - **Trigger Pattern**: Prefer direct API calls (`fetch(N8N_WEBHOOK_URL)`) from Next.js routes. Both the Yoco webhook and Reconcile worker explicitly trigger n8n.

## 🚨 Critical Constraints & Gotchas
- **Webhook Gap**: (Resolved 2026-03-15) The Yoco webhook now explicitly triggers n8n after confirming bookings in the DB.
- **SAST Timezone**: All bookings are SAST (+02:00). All API routes use a local `createSASTTimestamp` helper to ensure consistency.
- **Auth**: Dual-Auth Gateway.
  - **Admin PIN**: `8821` (Used for manual UI and updates).
  - **Reconcile Secret**: Used for automated Cron jobs.

## 🚀 Dev Commands
- `npm run dev` - Start dev server.
- `scripts/001-013_*.sql` - Database migration path.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sssammyboyyy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sssammyboyyy)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
