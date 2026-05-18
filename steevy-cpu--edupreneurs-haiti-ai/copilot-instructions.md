## edupreneurs-haiti-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server at http://localhost:8080
npm run build        # Production build → dist/
npm run build:dev    # Development build
npm run preview      # Preview production build locally
npm run lint         # Run ESLint (TypeScript + React)
```

There is no test suite configured — testing is manual.

## Environment

Copy `.env` and set the following variables:
```
VITE_SUPABASE_URL=https://<project>.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=<anon key>
VITE_SUPABASE_PROJECT_ID=<project id>
```

Supabase Edge Functions use server-side secrets configured in the Supabase Dashboard (not in `.env`): `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ELEVENLABS_API_KEY`, `RESEND_API_KEY`, `BAZIK_API_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `INTERNAL_SECRET`, `SUPABASE_SERVICE_ROLE_KEY`.

## Architecture

**Edupreneurs Haiti** is a French-language AI-powered educational platform for Haitian students (grades 7–Baccalauréat), deployed at mon-edupreneur.com.

### Stack
- **Frontend**: React 18 + TypeScript, Vite 5, Tailwind CSS, shadcn/ui + Radix UI, Framer Motion
- **State/Data**: TanStack Query v5 (5min stale, 30min GC, no window-focus refetch), React Router v6
- **Backend**: Supabase (PostgreSQL with RLS, 88 Deno edge functions)
- **AI**: Google Gemini 2.5 Flash/Pro + OpenAI GPT via Lovable Gateway
- **Mobile**: Capacitor v8 for iOS/Android builds

### Frontend Structure

```
src/
├── App.tsx              # Route definitions (52 pages, all lazy-loaded)
├── main.tsx             # Entry: Sentry init, service worker, React mount
├── shell/               # AppShell.tsx (persistent layout), sidebar, bottom nav, FloatingLayer
├── providers/           # Providers.tsx — 9 stacked context providers
├── contexts/            # SessionAuth, Network, Theme, Realtime, etc.
├── pages/               # Route-level components (all lazy via lazyWithRetry)
├── components/          # Feature-based: auth, course, lesson, jude, chess, community, etc.
├── hooks/               # 66 custom hooks — useOptimizedQuery, useNetworkAware*, useRealtime*
├── utils/               # Utilities: supabaseClient, lazyWithRetry, imageOptimization, networkDetection
├── integrations/supabase/ # client.ts + auto-generated types.ts (do not edit manually)
├── data/                # Static curriculum data, exam data, constants
└── types/               # TypeScript types (database.ts, api.ts, domain types)
```

**App Shell Pattern**: `AppShell.tsx` is the persistent shell (sidebar, bottom nav, FloatingLayer with Jude FAB and music player) that never unmounts. All pages render inside it.

**Lazy Loading**: Every page uses `lazyWithRetry()` from `src/utils/lazyWithRetry.ts` to handle stale chunk errors after deployments.

### Backend Structure

**Database**: PostgreSQL with 95 tables, RLS on all tables, 50+ database functions, 287 migrations. Key domains: users/profiles, content (lessons, quizzes, activities), gamification (gold, streaks, leaderboard), exams, quiz battles, chess, payments, community, notifications, admin.

**Edge Functions** (`supabase/functions/` — 88 Deno functions):
- AI generation uses a **job-based system** via the `ai_generation_jobs` table — prefer this over direct function calls for long-running generation
- Main job processor: `process-ai-job`
- AI tutors: `jude-ai-tutor`, `math-ai-tutor`, `francais-ai-tutor`, `bac-philosophy-tutor`, and 7 more
- Payments: MonCash (`moncash-*`), NatCash (`natcash-*`), Stripe (`stripe-*`)
- Email: `send-confirmation-email`, `send-welcome-email`, and 8 more via Resend

### Performance (3G-First)

The app is aggressively optimized for Haiti's 3G/slow connections:
- All pages are code-split and lazy-loaded
- Images use JPEG 0.65 quality, WebP/AVIF formats, avatars capped at 256×256
- `NetworkContext` detects 2G/slow-2G and disables animations
- Lesson audio is pre-generated and stored in Supabase Storage — never real-time TTS
- Chunk warning limit: 300KB
- TanStack Query cache: 5min stale, 30min GC, no window-focus refetch

### Authentication

1. Signup → email + password → profile creation → 6-digit email verification
2. Login → device fingerprint check → session creation
3. New device triggers a verification challenge (`user_trusted_devices` + `device_verification_challenges`)
4. Sessions are Supabase JWTs enforced via RLS on all tables

### Supabase Types

`src/integrations/supabase/types.ts` is auto-generated — never edit manually. Regenerate with:
```bash
npx supabase gen types typescript --project-id <project-id> > src/integrations/supabase/types.ts
```

### Key Files

| File | Purpose |
|------|---------|
| `src/App.tsx` | All routes + provider stack |
| `src/shell/AppShell.tsx` | Persistent layout shell |
| `src/providers/Providers.tsx` | All 9 context providers composed |
| `src/integrations/supabase/client.ts` | Supabase client singleton |
| `vite.config.ts` | Build config with 3G optimizations |
| `supabase/config.toml` | Edge function JWT settings |
| `ARCHITECTURE.md` | Detailed architecture documentation |
| `DATABASE.md` | Full database schema reference |

---
> Source: [steevy-cpu/edupreneurs-haiti-ai](https://github.com/steevy-cpu/edupreneurs-haiti-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
