## ace-your-permit

> Ace Your Permit is a gamified driving test prep PWA for Illinois, built with Next.js 16, Supabase, and Tailwind CSS. Target user: anyone preparing for the Illinois permit test — from 15yr-olds taking it for the first time to adults new to IL or retaking. Design for the procrastinator regardless of age.

# AGENTS.md — Ace Your Permit (formerly DriveMaster)

## Project Overview
Ace Your Permit is a gamified driving test prep PWA for Illinois, built with Next.js 16, Supabase, and Tailwind CSS. Target user: anyone preparing for the Illinois permit test — from 15yr-olds taking it for the first time to adults new to IL or retaking. Design for the procrastinator regardless of age.

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.

Key constraints:
- **Fonts:** Cabinet Grotesk (display), DM Sans (body), JetBrains Mono (data)
- **Colors:** Blue (#2563EB) primary, Orange (#F97316) accent, Green (#10B981) success
- **Touch targets:** 44px minimum on mobile
- **Motion:** Respect `prefers-reduced-motion`. No parallax.
- **Dash tone:** Hype-beast friend. Never say "incorrect." Max 3 sentences. One emoji max.

## Tech Stack
- Next.js 16 (App Router) + TypeScript 5
- Tailwind CSS 4 + shadcn/ui + Radix UI
- Supabase (Auth, PostgreSQL, Realtime)
- Framer Motion for animations
- Zustand for state management
- TanStack React Query for data fetching

## Commands
- `npm run dev` — start dev server
- `npm run build` — production build (validates env vars first)
- `npm run lint` — run ESLint

---
> Source: [h3cz/ace-your-permit](https://github.com/h3cz/ace-your-permit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
