## pickle-master

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PickleMaster** is a pickleball club SaaS platform MVP. Club leaders create sessions, members join and pay, and the platform handles capacity, payments, and notifications. This repo is currently in the **pre-implementation / specification phase** — no source code exists yet.

Key specs:
- `prd-v-1-0-0.md` — product requirements, user stories, personas, success metrics
- `mvp-system-design-v-1-0-0.md` — tech architecture, DB schema, API design, state machines, concurrency strategy

## Tech Stack

- **Frontend:** Next.js (App Router) deployed on Vercel
- **Styling:** Tailwind CSS + shadcn/ui (from `@/components/ui`)
- **Backend:** Next.js API routes + Supabase Edge Functions
- **Database:** Supabase PostgreSQL with Row-Level Security (RLS)
- **Auth:** Supabase Auth (email/password, Google OAuth, LINE Login)
- **Payments:** TapPay or NewebPay (Taiwan-native TWD settlement)
- **Realtime:** Supabase Realtime (WebSocket) — roster and capacity updates only
- **Storage:** Supabase Storage (club covers, profile images)

## Architecture

### Domain Model (11 tables)
`users` → `profiles` → `clubs` → `club_memberships` / `membership_applications` → `sessions` → `session_registrations` ↔ `payment_transactions` → `refund_transactions` → `notifications` / `analytics_events`

### Critical Path
Session publish → member join → payment intent + booking hold → card tokenization → webhook verification → confirmed registration → realtime roster update → refund on cancellation

### State Machines
Four entities have explicit state machines defined in the system design: membership applications, sessions, registrations, and payments. Refer to `mvp-system-design-v-1-0-0.md` before modifying state transition logic.

### API Design
28 endpoints across four domains: clubs, sessions, payments, and webhooks. All endpoints require Zod validation at the route entry point.

## Development Rules

### Spec-Driven Development
1. Always start by reading `docs/SPEC.md`. If it doesn't exist, draft it first.
2. Before writing any code, output a "Step-by-Step Execution Plan."
3. Wait for human [ACK] before modifying more than 2 files.
4. Use Plan Mode (read-only) to explore the codebase before proposing changes.
5. Every feature must update `docs/CHANGELOG.md` upon completion.

### MVP Mindset
- "Perfect is the enemy of Shipped."
- Choose boring tech over experimental unless the experiment IS the product.
- If a feature takes more than 4 hours to implement, suggest a manual workaround or third-party SaaS instead.

### UI Rules
- Use shadcn/ui components from `@/components/ui`. Check if a component exists before creating one (`ls src/components/ui`). Install missing components with `npx shadcn@latest add [name]`.
- Use Tailwind utility classes exclusively — no custom CSS files.
- Use the `cn()` utility for conditional classes.
- Reference `designs/images/` for overall visual vibe and `designs/components/` when styling individual components.

### API Validation (Zod-First)
- Every API route must have a Zod schema for request body and query params.
- Validation must happen at the top of the handler function.
- Return consistent error shapes: `{ error: string, code: string, details?: any }`.
- Use `unknown` instead of `any` for untrusted input; use type guards.
- Database queries must use the Prisma/Drizzle client from `@/lib/db`.

### Agentic / Multi-Agent Work
- For large tasks, spawn a Background Reviewer Agent to check for regressions.
- Use MCP tools to fetch real-time documentation for external APIs.
- Never delete existing tests — update them or revert the change.
- Limit parallel file edits to 5 at a time to maintain context coherence.

## Open Architecture Decisions (Unresolved)

Before implementing affected features, confirm these with the user:
1. **Payouts:** Direct to club leaders or platform-as-merchant-of-record?
2. **Service fees:** Global platform config or per-club?
3. **Session fees:** Single flat rate or member-tier pricing?
4. **Email provider:** Resend, SendGrid, or Supabase-native?
5. **Reminder timing:** 24h before, 2h before, or both?

## MVP Scope

**In scope (P0):** Club management, session CRUD with capacity enforcement, member applications/approval, join + payment flow with refunds, user auth, realtime roster updates, email + in-app notifications, analytics event capture.

**Out of scope for MVP:** LINE Pay, push notifications, member self-cancel, waitlist automation, skill-level matching, recurring sessions, multi-language UI, in-app chat, mobile app.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Douglashwang82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
