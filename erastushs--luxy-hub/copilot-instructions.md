## luxy-hub

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

<!-- BEGIN:dashboard-conventions -->
# LuxyHub Dashboard Conventions

## App Router

- All pages use the App Router (`app/` directory).
- Pages are Server Components by default.
- Client Components must have `'use client'` at the top.

## Layout

- `/app/dashboard/layout.tsx` is a Server Component that validates session via `getCurrentUser()`.
- The sidebar and nav are Client Components rendered within the Server Component layout.

## Session Auth

- `getCurrentUser()` from `app/lib/auth/session-auth.ts` is the canonical way to get the authenticated user.
- `requireAuth()` is used when the operation must fail without a session.
- Profile is auto-provisioned on first login by `getCurrentUser()`.

## Ownership

- `creator_id` is always derived from the server session — never from client input.
- `assertScriptOwner()` and `getOwnedScript()` enforce per-resource ownership.
- Cross-account isolation: non-owned resources return null/404-style responses.

## Server Actions

- All mutations use Server Actions in `app/actions/`.
- Dashboard pages call service/repository functions directly from Server Components — no client-side Supabase, no `fetch()` to dashboard API routes.

## Tailwind v4

- CSS-first config through `app/globals.css`.
- No `tailwind.config.ts`.
- Design tokens in `@theme inline` block.

## Phase Rules

- Phase 4 work is polish/performance/docs only — no new features, no API changes, no schema changes.
- Phase 5 and beyond require Secure Script Delivery design first.
- License management begins only after loader integration requirements are finalized.
- Marketplace, paid scripts, and creator economy are deferred indefinitely.
<!-- END:dashboard-conventions -->

---
> Source: [erastushs/luxy-hub](https://github.com/erastushs/luxy-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
