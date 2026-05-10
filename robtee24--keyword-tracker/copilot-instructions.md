## keyword-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Seauto** — an SEO and digital marketing SaaS platform. React frontend + Vercel serverless API functions backed by Supabase (Postgres + Auth). The product manages projects (websites) and provides SEO audits, keyword tracking, blog generation, ad creation, social media content, video ads, and page building.

## Development Commands

```bash
npm run dev          # Start Vite dev server (port 5173, proxies /api to localhost:3000)
npm run build        # TypeScript check + Vite production build
npm run vercel-dev   # Run with Vercel CLI (needed for testing serverless functions locally)
```

Local dev requires `vercel dev` (or `npx vercel dev`) for the API functions to work — the Vite dev server only proxies `/api` requests. The Vercel CLI serves the `api/` directory as serverless functions.

There are no test suites or linting commands configured.

## Architecture

### Frontend (`src/`)
- **React 19 + TypeScript + Tailwind CSS** single-page app with React Router
- `App.tsx` is the main orchestrator — manages auth state, project selection, and view routing via a `currentView` state string (not URL-based routes for the authenticated app)
- Public marketing pages use React Router routes (`/features/*`, `/solutions/*`, `/pricing`)
- The authenticated app lives at `/app` and uses sidebar navigation with `View` type strings
- Three context providers wrap the authenticated app: `PlanProvider`, `CreditsProvider`, `BackgroundTaskProvider`

Key patterns:
- `authenticatedFetch()` from `src/services/authService.ts` attaches the Supabase JWT to all API calls
- API endpoint URLs defined centrally in `src/config/api.ts` (`API_ENDPOINTS`)
- Plan gating via `<PlanGatedView>` and `<PlanGatedAuditView>` wrapper components in App.tsx
- Component files are large — most views are single-file components in `src/components/`

### Backend (`api/`)
- **Vercel serverless functions** (plain JavaScript, not TypeScript)
- Each file exports a default `handler(req, res)` function
- Shared helpers prefixed with `_` (e.g., `_config.js`, `_connections.js`, `_credits.js`, `_plans.js`)

Authentication pattern used in API handlers:
```js
import { authenticateRequest } from '../_config.js';
const auth = await authenticateRequest(req);
if (!auth) return res.status(401).json({ error: 'Unauthorized' });
const userId = auth.user.id;
```

Key shared modules:
- `api/db.js` — singleton Supabase admin client via `getSupabase()`
- `api/_config.js` — `authenticateRequest()` validates Supabase JWT from Authorization header
- `api/_connections.js` — OAuth token management (Google, Meta) with auto-refresh
- `api/_credits.js` — AI credit system: balance checks, deductions, enforcement
- `api/_plans.js` — plan tiers (base/plus/managed_digital) with usage limits and feature flags
- `api/_projectAccess.js` — validates user has access to a project by siteUrl
- `api/_contextPrompt.js` — builds context strings for AI image/video generation prompts
- `api/_fal.js` — fal.ai client wrapper for image/video generation
- `api/_imageGen.js` — image generation abstraction (FLUX via fal.ai, Gemini)

### Database (Supabase)
- Migrations in `supabase/migrations/` (numbered `001_` through `018_`)
- Key tables: `projects`, `project_members`, `service_connections`, `user_plans`, `usage_tracking`, `ai_credits`, `ai_credit_transactions`, `page_audits`, `blog_articles`, `blog_opportunities`, `build_results`, `social_posts`, `video_ads`, `ad_creatives`, `brand_profiles`
- Auth handled by Supabase Auth (Google OAuth, Microsoft/Azure, email/password)

### External Services
- **Anthropic** (Claude) — audits, content generation
- **OpenAI** — SEO recommendations
- **Google** — Search Console API, Ads API, Gemini/VEO for video/image gen
- **fal.ai** — FLUX image gen, Kling video, LTX video, image editing
- **Meta** — Facebook/Instagram Ads OAuth

## Deployment

Deployed on **Vercel**. `vercel.json` configures:
- Serverless function timeouts (up to 300s for video generation)
- SPA rewrite: all non-API/asset routes fall through to `index.html`
- CORS headers on `/api/*`

## Conventions

- API functions are plain `.js` files (not TypeScript), frontend is TypeScript
- Environment variables: server-side use `process.env.*`, client-side use `VITE_*` prefix
- All plans currently resolve to `managed_digital` (Stripe integration pending)
- Credits system: balance of `-1` means unlimited
- Domain matching uses normalized root domain fallback (`_domainMatch.js`) throughout connections and project access

---
> Source: [robtee24/keyword-tracker](https://github.com/robtee24/keyword-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
