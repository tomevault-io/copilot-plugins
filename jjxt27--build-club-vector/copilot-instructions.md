## build-club-vector

> In-event toolkit for Agentic AI Build Week (AABW). Two modes: **Pulse** (discovery/networking) and **Line** (heads-down build). Built as a hackathon submission and a product the organisers could run for real.

# Vector — project guide

In-event toolkit for Agentic AI Build Week (AABW). Two modes: **Pulse** (discovery/networking) and **Line** (heads-down build). Built as a hackathon submission and a product the organisers could run for real.

**Read next:** `docs/ARCHITECTURE.md` (how it's built) and `docs/ROADMAP.md` (what's left + known issues). This file is the orientation; those have the detail.

## Stack
Next.js 14 (App Router, TypeScript) + Supabase (Postgres, Auth, Realtime, Row-Level Security) + Vercel. Unit tests in Vitest.

## Run it
```bash
npm install
cp .env.example .env.local      # fill in the two NEXT_PUBLIC_SUPABASE_* values
npm run dev                     # http://localhost:3000
npm run test                    # vitest (pure logic in lib/)
npm run build                   # production build
```
You need a Supabase project with the schema applied (`db/migrations/*.sql` then `db/seed.sql`, run in the Supabase SQL editor) and in `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=https://<ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<publishable / anon key>
```
Email magic-link sign-in works with no extra setup. Google/GitHub OAuth need provider apps registered + creds added in the Supabase dashboard (see `docs/ROADMAP.md`).

## Deploy gotchas (read before deploying — these cost real time blind)
1. **Vercel framework preset must be Next.js.** It's pinned in `vercel.json` (`{"framework":"nextjs"}`). If it ever reverts to another preset, every route 500s/404s even though the build is green.
2. **The Edge middleware must stay dependency-free.** `lib/supabase/middleware.ts` does NOT import `@supabase/ssr` — that import fails to initialise in Vercel's Edge runtime and 500s the whole site. The middleware is a cookie-presence auth gate only; real security is enforced by RLS, and the browser client auto-refreshes the session.
3. **Set `NEXT_PUBLIC_SUPABASE_URL` + `NEXT_PUBLIC_SUPABASE_ANON_KEY` in Vercel** (Production + Preview). They're inlined at build into the client bundle.
4. **Add the deployed URL to Supabase Auth → URL Configuration** redirect allow-list, or magic-link/OAuth redirects fail.
5. Deploys are done via the Vercel CLI (`vercel --prod`), not Git integration (the Vercel account lacks access to the GitHub org). `.vercelignore` keeps `.env*`, `docs`, and local scratch out of the upload.

## What's real vs mock (important context)
- **Real / persisted:** accounts (magic-link; OAuth when configured), profiles, saved schedule, deadline checklist, and the **Bottleneck Radar** feed (live across users via Supabase Realtime — this is the hero).
- **Mock / seeded:** the event calendar, the People directory's example attendees (real signed-in users appear alongside them), and the Maps (illustrative SVG).
- **AI:** runs on local-logic fallbacks (no token spend). `api/claude.js` is a server proxy stub, currently unused/keyless.

## Conventions
- **The whole look tunes from `lib/design/tokens.ts`** (mirrored as CSS vars in `app/globals.css`). Design system: "embedding field / architect plotter on paper" — paper `#EEF1F4`, ink `#14143C`, vector-blue accent `#2B2BF5`, Fraunces display + IBM Plex Sans/Mono. Build UI from the primitives in `components/ui/*` + `components/shell/*`.
- The event clock is a deliberate **user-controlled simulation** (`lib/hooks/useSimClock.tsx`), not real time — so "now"/countdowns can be demoed at any moment. Do not replace it with real time.
- Plain, factual copy. No em-dashes in user-facing strings.
- Commit/push only when asked; this branch is `rebuild-v0`.

---
> Source: [jjxt27/Build-Club-Vector](https://github.com/jjxt27/Build-Club-Vector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
