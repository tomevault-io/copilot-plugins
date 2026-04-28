## composer

> Composer is a date and night-out itinerary generator for NYC. Users answer a short cascading questionnaire (occasion → neighborhood → budget → vibe) and receive a curated 3-stop evening itinerary structured as **Opener → Main → Closer**.

# CLAUDE.md — Composer

## Project Overview

Composer is a date and night-out itinerary generator for NYC. Users answer a short cascading questionnaire (occasion → neighborhood → budget → vibe) and receive a curated 3-stop evening itinerary structured as **Opener → Main → Closer**.

The product is built on a **hybrid curation model**: the venue database is human-curated by the founders (Reid and Adit), scored and assembled by weighted algorithm, and polished by the Claude API for copy voice. The human taste layer is the core differentiator — this is not a generic AI recommendation engine.

**Primary target: Mobile-responsive web.** Website first at onpalate.com/composer. iOS via Capacitor is Phase 2. Every UI decision should work on a phone screen first.

**Auth: Supabase email/password.** Users sign in or sign up on a single combined entry screen (`AuthScreen`) with email + password — the form tries `signInWithPassword` first and falls back to `signUp` on an "invalid / not found" error, so there's no explicit sign-in/sign-up toggle. Forgot-password flow uses `resetPasswordForEmail` with a redirect to `/auth/reset`. No OAuth providers. Profile and saved itineraries live in Supabase tables with RLS (`composer_users`, `composer_saved_itineraries`). The one exception to "no client persistence" is the page-to-page sessionStorage bridge between `/compose` and `/itinerary` — that's in-tab flight state, not user state.

---

## Tech Stack

- **Framework**: Next.js 14+ (App Router) + TypeScript
- **Styling**: Tailwind CSS
- **Database**: Supabase (PostgreSQL + Row Level Security)
- **AI**: Google Gemini 2.5 Flash (`gemini-2.5-flash`) — itinerary copy and voice
- **Weather**: OpenWeatherMap API — called per generation, not cached
- **Package Manager**: npm
- **Deployment**: Vercel
- **Mobile (Phase 2)**: Capacitor → iOS

---

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
GEMINI_API_KEY
OPENWEATHERMAP_API_KEY
```

Never hardcode these. Never commit `.env.local`. Always use `process.env.*` server-side and `NEXT_PUBLIC_*` only when the value is safe to expose to the client.

---

## Project Structure

```
app/
├── page.tsx                  # Landing page
├── compose/
│   └── page.tsx              # Questionnaire flow
├── itinerary/
│   └── page.tsx              # Composition output
└── api/
    └── generate/
        └── route.ts          # POST endpoint: weather + scoring + Claude → itinerary

components/
├── ui/                       # Base UI primitives (Button, OptionCard, ProgressBar)
├── providers/                # AuthProvider (context for user/profile/session)
├── landing/                  # Hero, CTA
├── home/                     # HomeScreen (signed-in landing with saved plans)
├── auth/                     # AuthScreen + ForgotPasswordScreen (email/password)
├── onboarding/               # OnboardingFlow (name + profile, session already active)
├── questionnaire/            # QuestionnaireShell, StepLoading, OptionCard
└── itinerary/                # CompositionHeader, StopCard, WalkConnector, ActionBar

lib/
├── supabase.ts               # Anon Supabase client for non-auth reads (venues)
├── supabase/browser.ts       # Browser auth-aware client (@supabase/ssr, cookie session)
├── supabase/server.ts        # Server auth-aware client for Route Handlers
├── auth.ts                   # Sign in / sign up / reset-password / profile helpers
├── scoring.ts                # Weighted venue scoring + itinerary composer
├── weather.ts                # OpenWeatherMap fetch + rain/snow classification
├── geo.ts                    # Haversine distance + Manhattan grid correction + Maps URL builder
├── claude.ts                 # Gemini API call + graceful fallback
└── sharing.ts                # URL param encode/decode for share links

config/
├── options.ts                # All questionnaire step definitions
└── prompts.ts                # Claude system prompt + generation prompt builder

supabase/
└── seed.sql                  # Schema + seed venues
```

---

## Database Schema

```sql
-- See supabase/migrations/20260419_venue_schema_v2.sql for the full DDL.
composer_venues (
  id uuid primary key,
  name text not null, neighborhood text not null, category text not null,
  price_tier int check (1-4),
  vibe_tags text[], occasion_tags text[], stop_roles text[],
  duration_hours int, outdoor_seating text ('yes'/'no'/'unknown'),
  reservation_difficulty int, reservation_url text, maps_url text,
  curation_note text, awards text, curated_by text, signature_order text,
  address text, latitude float not null, longitude float not null,
  active boolean, notes text, hours text, last_verified date,
  happy_hour text, dog_friendly boolean, kid_friendly boolean,
  wheelchair_accessible boolean, cash_only boolean,
  quality_score int default 7, curation_boost int default 0,
  created_at timestamptz
)
```

### Canonical Vibe Tags

Tag lists are generated from the Google Sheet via `npm run generate-configs`.
To add a tag: edit the sheet's Vibe Tags or Vibe Scoring Matrix tab, re-run the script.

**Scored tags** (mapped to questionnaire vibe selections, 35% weight):

| Tag | Maps to vibe |
|-----|-------------|
| `food_forward`, `tasting`, `dinner`, `bistro` | food_forward |
| `cocktail_forward`, `wine_bar`, `speakeasy`, `drinks` | drinks_led |
| `activity`, `comedy`, `karaoke`, `games`, `bowling` | activity_food |
| `walk`, `gallery`, `bookstore`, `market`, `park` | walk_explore |

**Cross-cutting tags** (valid, not scored by vibe):
`romantic`, `conversation_friendly`, `group_friendly`, `late_night`, `casual`, `upscale`, `outdoor`, `classic`, `iykyk`, `lunch`, `cash_only`, `reliable`

### Neighborhood Slugs

Always snake_case. Must match exactly across the DB, `config/neighborhoods.ts`, and the venue sheet:
`west_village`, `east_village_les`, `soho_nolita`, `williamsburg`, `midtown_hells_kitchen`, `upper_west_side`

### Auth Tables (20260415 migration)

Both have RLS on with `auth.uid()`-scoped policies. The anon client can only see the signed-in user's own rows.

```sql
composer_users (
  id uuid primary key references auth.users(id) on delete cascade,
  name text not null,
  context text,
  drinks text,
  dietary text[] default '{}',
  favorite_hoods text[] default '{}',
  is_admin boolean not null default false,
  created_at timestamptz
)

composer_saved_itineraries (
  id uuid primary key,
  user_id uuid references composer_users(id) on delete cascade,
  title text, subtitle text,
  occasion text, neighborhoods text[], budget text, vibe text, day text,
  stops jsonb, walking jsonb, weather jsonb,
  created_at timestamptz
)
```

Admin access is granted by setting `is_admin = true` on the `composer_users` row directly in Supabase — never via the app. One-liner:

```sql
update composer_users set is_admin = true where id = (
  select id from auth.users where email = 'someone@example.com'
);
```

The existing RLS policy (`auth.uid() = id`) means each session can only read its own profile row, so admin status isn't leaked between users. `AuthProvider` exposes it as `useAuth().isAdmin`.

---

## Architecture Principles

### API Route for Generation
All itinerary generation happens server-side in `app/api/generate/route.ts`. The client POSTs questionnaire answers and receives a complete itinerary. The client never calls Supabase, OpenWeatherMap, or Claude directly.

### Supabase Access Splits by Trust Boundary
- **Anon reads of public data (venues):** `lib/supabase.ts` via `getSupabase()` — no cookie session, fine for Route Handlers that don't need `auth.uid()`.
- **Client-side user-scoped reads/writes (profile, saved itineraries):** `lib/supabase/browser.ts` via `getBrowserSupabase()`. Session lives in cookies so the server can see it.
- **Server-side user-scoped reads:** `lib/supabase/server.ts` via `getServerSupabase()` inside Route Handlers. This is how `/api/generate` resolves the signed-in user's profile for personalization and hard filters.
- RLS enforces row visibility. Components calling `getBrowserSupabase()` is allowed — RLS gates the data, not the import boundary.

### Scoring Logic Lives in `lib/scoring.ts`
The weighted scoring algorithm and itinerary composer are isolated here. Do not inline scoring logic in the API route. If scoring behavior needs to change, it changes in one place.

### Weather is Stateless
`lib/weather.ts` fetches current NYC conditions per request. There is no caching layer. This is intentional — itineraries should reflect actual current conditions.

### Claude as Polish Layer, Not Core Logic
The Claude API call in `lib/claude.ts` is a copy enhancement step, not the core logic. The scoring and venue selection happen first. Claude receives the selected venues and writes the composition header and personalizes curation notes. If the Claude call fails, `lib/claude.ts` falls back gracefully to the raw `curation_note` from the database — the itinerary still renders.

---

## Scoring System

Vibe match scoring in `lib/scoring.ts` uses **exact canonical tag matching** via set intersection. No substring matching, no fuzzy matching.

```typescript
const VIBE_TAGS: Record<string, string[]> = {
  "food_forward":  ["food_forward", "tasting", "dinner", "bistro"],
  "drinks_led":    ["cocktail_forward", "wine_bar", "speakeasy", "drinks"],
  "activity_food": ["activity", "comedy", "karaoke", "games", "bowling"],
  "walk_explore":  ["walk", "gallery", "bookstore", "market", "park"],
  "mix_it_up":     [],  // empty = no vibe filter, all venues score equally on this dimension
};
```

Scoring tiers:
- 2+ tag hits = full 35pts
- 1 hit = 25pts
- 0 hits = 10pts base (venue can still appear if other factors score high)

**Never change this to substring or fuzzy matching.** Fragile matching was the original bug — it's been fixed.

### Weighted Score Breakdown

| Factor | Weight |
|--------|--------|
| Vibe match | 35% |
| Occasion fit | 15% |
| Budget fit | 15% |
| Location fit (walkable cluster) | 10% |
| Time fit | 10% |
| Quality signal (curation tier) | 10% |
| Curation boost (reid/adit picks) | 5% |

### Itinerary Composition
The composer picks the best **combination** not the top 3 individual scores. Priority factors:
- Geographic clustering (all stops walkable, ideally <15 min between each)
- Category variety (no two stops the same category)
- Pacing (light opener → heavier main → wind-down closer)
- Budget distribution

### Progressive Filter Relaxation
If filters return too few venues (<3 candidates per role), the scorer relaxes constraints in order: neighborhood → budget → occasion. It never returns an empty itinerary. Log a warning when relaxation triggers — it signals a database coverage gap.

### Plan B
For each flexible stop (Opener and Closer), a backup venue is pre-generated at composition time. Same hard filters, different category from primary. Fixed stops (Main) do not get Plan B.

---

## Weather Gate

OpenWeatherMap is called in `lib/weather.ts` at generation time. Classification:
- `rain` or `snow` → eliminate `outdoor_seating = true` venues
- Extreme temp (< 32°F or > 90°F) → same penalty as rain
- Clear → no adjustment

Surface a weather note in the composition header only when conditions affected the output. Don't show weather info if it didn't matter.

---

## Questionnaire Flow

Defined in `config/options.ts`. Five steps, each with an explicit "Next →" button:
1. **Occasion** — first_date | dating | couple | friends | solo
2. **Neighborhoods** — pick up to 3 from borough-grouped pills
3. **Budget** — casual | nice_out | splurge | all_out | no_preference
4. **Vibe** — food_forward | drinks_led | activity_food | walk_explore | mix_it_up
5. **When** — day (7-day pills + custom date picker) + duration (keep it short / enjoy the moment / open-ended)

No auto-advance — every step requires an explicit button tap to proceed. Occasion pre-fills from `profile.context` via `CONTEXT_TO_OCCASION`. Neighborhoods pre-fill from `profile.favorite_hoods`.

---

## Gemini API

Model: `gemini-2.5-flash`
Max tokens: 1000

System prompt (from `config/prompts.ts`):
```
You are the voice of Composer, a curated NYC date night app founded by two people 
known for their taste in the city. Write in a warm, confident, first-person plural 
voice. You are opinionated. Say "this is the move" not "you might enjoy." 
Keep all copy concise. Never hedge. Never list more than you need to.
```

**Do not change the system prompt without discussing with the founders.** Brand voice is intentional.

The Gemini call always has a graceful fallback. If it throws or times out, use the raw `curation_note` from the DB. Never block itinerary rendering on a Gemini API failure. (Note: the implementation still lives in `lib/claude.ts` — that's the filename only, not the underlying API.)

---

## Design System

### Typography
- **Display / venue names / titles**: Playfair Display (serif, Google Fonts)
- **UI / body / labels**: DM Sans (sans-serif, Google Fonts)

### Colors
```
Background:         #FAF8F5  (warm off-white)
Primary accent:     #6B1E2E  (deep burgundy)
Secondary accent:   #1E3D2F  (forest green)
Text primary:       #1A1A1A
Text secondary:     #6B6B6B
```

Never use generic purple gradients, white backgrounds, or Inter/Roboto. The aesthetic is editorial and warm, not startup.

### Principles
- Mobile-first always
- No decorative gradients
- Staggered entrance on composition output (Opener → Main → Closer sequential reveal)
- Loading states feel intentional — never a bare spinner
- Touch targets minimum 44x44px

---

## Coding Standards

### TypeScript
- Strict mode on. No `any` types. No `ts-ignore`.
- Define types in `types/` for shared data shapes. Inline types for local-only shapes.
- All API route handlers must type their request body and response.
- Supabase query results must be typed — use generated types or explicit casting, never implicit `any`.

### React / Next.js
- App Router only. No Pages Router patterns.
- Server Components by default. Add `"use client"` only when necessary (interactivity, hooks, browser APIs).
- No God components. If a file exceeds 250 lines, split it.
- One component per file. File name matches component name in kebab-case (`stop-card.tsx` exports `StopCard`).
- No inline styles. Tailwind only. If a custom value is needed more than once, extract to a CSS variable.

### Data Fetching
- Server Components fetch directly for public data. Route Handlers use `getServerSupabase()` for auth-scoped reads.
- Client-side state is minimal — questionnaire answers in `useState`, itinerary result passed via URL params or sessionStorage (page-to-page in-tab bridge only).
- Client components that need the current user read from `useAuth()` rather than fetching auth state themselves.
- `useEffect` for data fetching is acceptable for user-scoped client-side reads (e.g. HomeScreen's saved plans list) since those fire only on mount and need the session cookie.

### Error Handling
- All API routes return typed error responses with appropriate HTTP status codes.
- Client components handle error states visibly — never silently swallow errors.
- The Claude fallback in `lib/claude.ts` must be tested — do not remove it.

### File Naming
- Components: `PascalCase.tsx`
- Utilities / libs: `camelCase.ts`
- Config files: `camelCase.ts`
- All exported functions and components use named exports. No default exports except Next.js page files.

### Imports
Order: React → Next.js → third-party → internal (`@/lib`, `@/components`, `@/config`, `@/types`)

### Git Commits
Format: `type(scope): description`
Types: `feat`, `fix`, `chore`, `refactor`, `style`, `docs`

**Keep commit messages concise — one line only.** No multi-line bodies, no bullet lists, no co-author trailers. The git history should stay scannable. If a change is so large it can't be summarized in one line, that's a signal to split it into multiple commits.

Examples:
```
feat(scoring): add progressive filter relaxation for thin neighborhoods
fix(weather): handle OpenWeatherMap timeout gracefully
chore(venues): add 12 new West Village venues to seed
```

**Never run `git commit`, `git push`, or `git add`.** When a task is complete, provide the suggested commit message in the format above and stop. The developer runs all git commands manually.

---

## Performance Rules

- No animations that block interaction or feel slow on a mid-range Android device
- Google Maps export URL is constructed client-side in `lib/geo.ts` — no API call needed
- OpenWeatherMap call happens server-side in the API route — never from the client
- Do not add new npm dependencies without justification. Prefer what's already in the project.

---

## Venue Database Rules

- **Never edit venues directly in the DB.** The Google Sheet is the single source of truth. All changes go through the sheet → import pipeline below.
- All taxonomy slugs (neighborhoods, vibes, occasions, budgets, stop roles) use snake_case to match the sheet's dropdown validation.
- `active = false` hides a venue from scoring. Use this instead of deleting records.
- The `notes` column in the Google Sheet is internal only — stored in the DB but not surfaced in the app.

### Updating Venue Data

When you add, edit, or remove venues in the Google Sheet:

```bash
# 1. Export the sheet as xlsx to the repo
#    File → Download → .xlsx → save to docs/composer_venue_sheet_curated.xlsx

# 2. Regenerate scoring configs (if you changed any reference tab —
#    Vibe Tags, Neighborhood Groups, Stop Roles, Budget Tiers, etc.)
npm run generate-configs

# 3. Generate the import SQL from the Venues tab
python3 scripts/import_venues.py --out /tmp/import.sql

# 4. Apply to Supabase (replace the connection string with yours)
#    Or pipe the SQL through the Supabase SQL editor.
psql "$DATABASE_URL" < /tmp/import.sql

# 5. Verify
#    Hit /api/health or generate an itinerary to confirm.
```

The import is idempotent — re-running upserts via the `(LOWER(name), neighborhood)` unique index. Existing rows update; new rows insert; nothing deletes (set `active = false` in the sheet to hide a venue).

### Updating Scoring Configs Only

If you changed a reference tab (e.g. added a vibe tag, tweaked neighborhood groups) but didn't touch venue data:

```bash
npm run generate-configs   # regenerates src/config/generated/*.ts
npx tsc --noEmit           # verify types still pass
# Commit the generated files + deploy
```

No import needed — the generated configs are committed to git and take effect on the next deploy.

---

## What NOT To Do

- Don't call anon Supabase (`lib/supabase.ts`) from client components. For user-scoped data use `getBrowserSupabase()`.
- Don't add AI-generated venues to the database. Every venue must be human-verified.
- Don't change the vibe tag matching from exact to substring/fuzzy. This was a deliberate fix.
- Don't change the Claude system prompt without founder approval.
- Don't add loading states that feel like the app is doing more work than it is.
- Don't use `any` types or `ts-ignore`.
- Don't add new neighborhood slugs without updating `config/options.ts`, the venue sheet Reference tab, and the DB validation simultaneously.
- Don't use `localStorage` for user state. Profile and saved plans live in Supabase. `sessionStorage` is acceptable only for page-to-page in-tab flight state.
- Don't add OAuth providers. The auth model is email/password only.
- Don't add features that aren't in the PRD without flagging them first. Scope creep kills MVPs.
- Don't assume desktop-first. Mobile is the primary surface.
- Don't run `git commit`, `git push`, or `git add`. Provide the commit message and let the developer run it.

---

## Product Context

**Founders:** Adit and Reid
**Platform:** onpalate.com/composer (part of the Palate brand alongside Pour Decisions)
**Launch target:** Columbia Business School community (NYC)
**MVP success metric:** 50 compositions generated in week 1, 200+ by week 4

The product works because of the curation layer, not the technology. Reid and Adit are known in the CBS community as the go-to people for NYC date and dinner recommendations. They host weekly dinner reservations around the city. The app is the productization of that reputation.

When in doubt about a product decision, ask: does this make the output feel more trustworthy and opinionated, or less?

---

## Phase 2 (Not Building Yet)

- Community venue submissions
- Google Places / Resy / OpenTable live sync
- Native reservation booking
- Implicit preference learning
- iOS app via Capacitor
- Monetization (venue partnerships, premium tier)
- MAKE MONEY

---
> Source: [aryaadit/composer](https://github.com/aryaadit/composer) — distributed by [TomeVault](https://tomevault.io/claim/aryaadit).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
