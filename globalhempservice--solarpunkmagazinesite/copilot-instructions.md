## solarpunkmagazinesite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## !! CRITICAL WORKFLOW RULES !!

- **NEVER push to GitHub unless the user explicitly says "push to GitHub"**
- **NEVER commit directly to `main`** — always work on a feature branch: `git checkout -b fix/description`
- Work locally against the production Supabase database (dewii)
- localhost:3000 is the dev environment; production is mag.hempin.org (Netlify, auto-deploys from `main`)

## Commands

```bash
npm run dev      # Start dev server on port 3000
npm run build    # Production build to 'build' directory
```

No lint or test scripts configured. Manual helpers: `src/test-server-locally.sh`, `src/test-build.sh`.

## Environment

The app uses hardcoded Supabase credentials in `src/utils/supabase/info.tsx` pointing to the production project (`dhsqlszauibxintwziib`). There is no `.env` switching between environments — localhost and production both hit the same Supabase project.

Edge functions are deployed to production via:
```bash
supabase functions deploy make-server-053bcd80 --project-ref dhsqlszauibxintwziib --use-api
```

The deploy symlinks live in `supabase/functions/make-server-053bcd80/` (pointing to `src/supabase/functions/server/`).
Note: the two `index.tsx` files are **hardlinked** (same inode) — editing either one edits both.

## RSS Auto-Publish

The MAG auto-publishes RSS articles daily via `POST /rss/daily-sync`. Setup status:

- ✅ Edge function endpoint built (`/rss/daily-sync`, accepts `x-cron-secret` header)
- ✅ Keyword-based auto-categorisation (6 categories + Eco Innovation fallback)
- ✅ Per-feed `default_category` override (set when subscribing to a feed)
- ⏳ **Supabase pg_cron NOT yet configured** — daily sync does NOT run automatically yet

To activate daily auto-publish, run in Supabase SQL editor:
```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;
CREATE EXTENSION IF NOT EXISTS pg_net;

SELECT cron.schedule(
  'rss-daily-sync',
  '0 6 * * *',  -- 6am UTC = 2am ET = 7am CET
  $$
  SELECT net.http_post(
    url := 'https://dhsqlszauibxintwziib.supabase.co/functions/v1/make-server-053bcd80/rss/daily-sync',
    headers := '{"Content-Type": "application/json", "x-cron-secret": "YOUR_CRON_SECRET_HERE"}'::jsonb,
    body := '{}'::jsonb
  );
  $$
);
```

Also set `CRON_SECRET` in Supabase Dashboard → Edge Functions → Secrets (match the value above).
To change time: `SELECT cron.unschedule('rss-daily-sync');` then reschedule.

## Architecture

React 18 + TypeScript + Vite SPA. No client-side router — navigation is a `currentView` state variable in `App.tsx` that switches between named views (`feed`, `dashboard`, `editor`, `article`, `admin`, `swipe`, `profile`, etc.).

### App.tsx (root)

Single source of truth for all state (~2000+ lines):
- **Auth**: `isAuthenticated`, `accessToken`, `userId`, `userEmail`, `displayName`
- **Content**: `articles[]`, `userArticles[]`, `selectedArticle`, `searchQuery`
- **User progress**: `userProgress` (XP, level, streak, NADA points), `userBadges`
- **UI/routing**: `currentView`, modal states, drawer states

### Mini-Apps System

Self-contained "mini-apps" with isolated branding, each wrapped by `MiniAppContainer.tsx`:
- **HEMP MAG** — article browser (purple)
- **HEMP SWIPE** — tinder-style discovery (red)
- **HEMP PLACES** — geographic directory (emerald)
- **HEMP SWAP** — C2C barter marketplace (cyan)
- **HEMP FORUM** — community discussions (indigo)
- **HEMP TERPENE** — cannabinoid hunter game (orange)
- **HEMP GLOBE** — 3D world map (teal)

Metadata in `src/config/mini-apps-metadata.tsx`. Types in `src/types/mini-app.ts`. Components under `src/components/mini-apps/`.

### Backend (Supabase Edge Functions)

Hono.js TypeScript API at `src/supabase/functions/server/`. Key files:
- `index.tsx` — main router, `requireAuth` middleware, signup/login handlers
- `rss_parser.tsx` — RSS feed ingestion
- `discovery_routes.tsx` — matched article recommendations
- `swag_routes.tsx` / `swap_routes.tsx` — marketplace
- `places_routes.tsx` — geographic CRUD
- `wallet_security.tsx` — NADA point transaction validation

The function is deployed with `--no-verify-jwt` — the Supabase runtime does NOT pre-validate JWTs. `requireAuth` middleware handles all JWT validation via `supabaseAuth.auth.getUser(token)`.

### Key DB Tables (production)

- `profiles` — created by signup handler (id, email, name, accepted_terms, marketing_opt_in)
- `user_profiles` — created by `on_auth_user_created` trigger (id, display_name, avatar_url)
- `user_progress` — lazy-created by edge function on first login
- `wallets` — lazy-created on first point exchange
- `user_progress_complete` — VIEW joining progress/achievements for gamification

### UI Components

`src/components/ui/` — shadcn-style wrappers around Radix UI. Use these, not raw Radix imports.

### Path Aliases

`@` → `/src` (vite.config.ts + tsconfig.json)

---

## Known Issues & Technical Debt

> See **AUDIT.md** in the project root for the full detailed audit report with context, examples, and rationale.

### Immediate (low risk — do these first, one branch per fix)
1. **Delete dead welcome page variants** — `ModernWelcomePage`, `PremiumWelcomePage` and other unused files in `src/components/welcome/`
2. **Remove Figma Make artifacts** — `Code-component-*.tsx` files in `src/public/_redirects/` and the duplicate `src/src/` directory
3. **Replace `confirm()` dialogs** — native browser confirm() used for destructive actions, replace with shadcn `AlertDialog`
4. **Strip console.logs from production** — 132 in App.tsx alone, add vite plugin to strip in prod builds

### Short-term
5. **Add router (wouter)** — replace `currentView` string state with proper URL routing, enables deep linking and back/forward
6. **Add `React.lazy()`** — wrap all 8 mini-app components for code splitting, nothing should load until needed
7. **Replace window globals** — `(window as any).__swapOpenAddModal` pattern in 4+ places, replace with React Context
8. **Add error boundaries** — wrap each mini-app so a WebGL crash doesn't white-screen the entire app

### Medium-term
9. **Break up App.tsx** — extract `AuthProvider` and `UserProvider` contexts, App.tsx is 2000+ lines with 42 hooks
10. **Clean up server monolith** — `src/supabase/functions/server/index.tsx` is 9062 lines, split dead routes out

---

## Git Workflow

```bash
# Before ANY change:
git checkout -b fix/description

# After testing on localhost:3000:
git add .
git commit -m "fix: description"

# Only push when user explicitly says to push to GitHub
```

**Never merge to main without user approval. Never auto-push.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/globalhempservice) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
