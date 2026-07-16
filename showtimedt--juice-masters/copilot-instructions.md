## juice-masters

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Juice Tour — fantasy golf for the majors

Friends form leagues and draft 8 golfers (one per tier) before each men's major.
Live ESPN scoring ranks teams by their best 5 golfer scores; a season leaderboard
runs across all four majors. Live in production on Vercel since the 2026 U.S. Open.

**Daniel (the owner) is not an engineer.** Explain things in plain English,
keep components small and reusable, and never bury styling decisions in code.

## Stack

- Next.js 16 (App Router) + React 19 + TypeScript, Tailwind CSS 4
- Neon Postgres via `@neondatabase/serverless` (tagged-template SQL, no ORM)
- NextAuth v5 beta — email/password (bcryptjs) + Google OAuth, JWT sessions
- Deployed on Vercel; ESPN + The Odds API for golf data

## Commands

- `npm run dev` — local dev server (needs `.env.local`, see below)
- `npm run build` — production build; run before merging
- `npm run lint` — ESLint
- `npx vitest run` — unit tests (scoring/season logic in `src/lib/__tests__/`)
- `npx tsx scripts/migrate.ts` — create/alter database tables (idempotent; the
  schema lives in this file — there is no migrations folder)

## Map of the code

| Area | Where |
|------|-------|
| Scoring (best-5, missed-cut +10, tiebreaker) | `src/lib/scoring.ts` |
| Season standings + missed-major penalty | `src/lib/season.ts` |
| Which major to show, live states | `src/lib/tournament-state.ts` |
| Tournament list, dates, venues, **per-major themes** | `src/lib/tournaments.ts` |
| ESPN scoreboard parsing | `src/lib/espn.ts` |
| Authorization helpers (use these in every API route) | `src/lib/authz.ts` |
| Input validation limits/helpers | `src/lib/validate.ts` |
| Database schema | `scripts/migrate.ts` |
| API routes | `src/app/api/{auth,draft,leagues,scores,me}/` |
| Pages | `src/app/` — league pages under `league/[slug]/` |
| Components | `src/components/` |

## Conventions

- **Colors are never hardcoded in components.** App-wide tokens live in
  `src/app/globals.css` (the "Dawn" theme: dark green surfaces, gold brand,
  Cormorant/Jost/JetBrains Mono fonts). Per-major colors live in the
  `TournamentTheme` objects in `src/lib/tournaments.ts`. Use Tailwind classes
  backed by those variables (`bg-surface`, `text-ink`, `text-gold`, `text-under`,
  `text-over`) or `var(--...)` in inline styles.
- Every mutating API route starts with `requireUser()` /
  `requireLeagueCommissioner()` / `requireLeagueMember()` from `src/lib/authz.ts`
  and returns `authzError(...)` on failure.
- Validate request bodies with helpers in `src/lib/validate.ts`; add new limits
  there, not inline.
- Unfinished backend work is marked `TODO(backend)` with an explanation; UI for
  missing backends must be honest (disabled buttons, "coming soon"), never a
  silent no-op.

## Environment (`.env.local`, never committed)

`DATABASE_URL` (Neon), `AUTH_SECRET`, `AUTH_GOOGLE_ID`/`AUTH_GOOGLE_SECRET`
(optional), `ODDS_API_KEY` (The Odds API — the **primary** source for draft
fields; ESPN athlete ordering is NOT odds-ranked, see the June 2026 scrambled-
tiers fix), `ADMIN_PASSWORD`.

## The season

Masters (April) → PGA Championship (May) → U.S. Open (June) → The Open (July).
Standings default to the most recent major for a week after it ends, then the
season view. Dates/venues per year are configured in `src/lib/tournaments.ts`.

---
> Source: [ShowtimeDT/juice-masters](https://github.com/ShowtimeDT/juice-masters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
