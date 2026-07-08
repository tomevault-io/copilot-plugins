## surpry

> Mobile-first web app to organize secret birthday gift coordination in groups.

# AGENTS.md

## Project
Mobile-first web app to organize secret birthday gift coordination in groups.

## Stack
React + Vite + JavaScript
Supabase (Auth, Postgres, Storage, Realtime, RLS)
Vercel

## Core product rules
- The birthday user must never see hidden event data.
- No in-app payment processing.
- Support multiple expenses per event.
- Optimize for mobile first.
- Keep components small and reusable.
- Use feature-based folder structure.

## Commands
- npm install
- npm run dev
- npm run build

## Engineering conventions
- Prefer simple components and hooks.
- Keep server state separated from UI state.
- Avoid premature abstraction.
- Keep forms explicit and readable.
- Add comments only when logic is not obvious.

## Design system
- Colors from CSS tokens in `src/styles/tokens.css` — use Tailwind aliases (primary, surface, bg, text-muted, etc.)
- Typography: Plus Jakarta Sans, font-size via Tailwind (text-sm / text-base / text-lg / text-2xl)
- Radius: rounded-2xl (cards), rounded-3xl (large cards), rounded-full (pills/avatars)
- Shadows: shadow-card (elevation), shadow-float (teal glow for primary actions)
- Icons: Material Symbols Outlined, 1rem/1.25rem/1.4rem/1.5rem sizes. Use fontVariationSettings "'FILL' 1" for filled icons.

## UI components (src/components/ui/)
- `Button` — variants: primary / secondary / ghost / danger. Sizes: sm / md / lg / pill (full-width rounded-full CTA)
- `Avatar` — props: name, url, className, ring (teal gradient ring), badge (overlay slot for edit icons etc.)
- `Card` — rounded-3xl with shadow-card
- `ProgressBar` — props: value (0-100), label, rightLabel
- `AvatarStack` — props: users[{name, avatar_url}], max
- `CountdownTimer` — props: targetDate (ISO string), renders DD/HRS/MIN/SEG chips
- `StatusBadge` — maps event/share statuses to colored badges
- `FormField`, `Input`, `Select`, `TextArea` — standard form primitives

## Layout components (src/components/layout/)
- `AppShell` — wraps page content with BottomNav; props: activeTab, header, hideNav, className
- `BottomNav` — 4 tabs (inicio / grupos / eventos / perfil) with pill active-state highlight
- `PageHeader` — sticky header; props: title, subtitle, backTo, action (right slot)

## Current build phase
Phase 1–7: Auth, profiles, groups, wishlist, secret events, expenses, reimbursements, and cron-job auto-creation are implemented and core functional.
Phase 8+: Notifications UI, PWA configuration, Realtime subscriptions, and performance polish are still pending.

## Done means
- Code runs locally.
- No broken routes.
- No unused imports.
- Any new data access respects RLS assumptions.
- Any new feature includes empty/loading/error states.

---
> Source: [JusmeJr93/surpry](https://github.com/JusmeJr93/surpry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
