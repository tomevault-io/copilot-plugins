## lookforme

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LookForMe (LFM-DOM) is a NYC-focused mobile web app for location-based missed connections. Users post about meaningful encounters at specific places (restaurants, parks, transit) and others can find, match, and connect privately via a verification gate. Think Hinge meets a map, not Craigslist missed connections.

There are two implementations in this repo:
1. **`app/`** — The modern Vite + React + TypeScript app (active development)
2. **`old/LFMJesse.html`** — The original single-file prototype (~4,500 lines, `React.createElement` without JSX)

## Commands

### Modern app (`app/`)

```bash
cd app
npm install          # Install dependencies
npm run dev          # Dev server on http://localhost:5173
npm run build        # TypeScript check + production build (tsc -b && vite build)
npm run lint         # ESLint
npm run test         # Run tests (vitest)
npm run preview      # Preview production build
```

Requires `VITE_MAPBOX_TOKEN`, `VITE_SUPABASE_URL`, and `VITE_SUPABASE_ANON_KEY` in `app/.env` (see `app/.env.example`). Search also requires `OPENAI_API_KEY` set as a Supabase secret (not a Vite env var).

### Prototype (legacy, `old/`)

```bash
open old/LFMJesse.html                # Open directly in browser
cd old && python3 -m http.server 8000 # Serve locally (needed for Google Maps)
```

No build step. Loads React 18 + Tailwind from CDN. Google Maps API key is embedded in the HTML.

## Architecture (Modern App)

**Stack**: Vite 7, React 19, TypeScript 5.9, Tailwind 4, Mapbox GL 3.18, Zustand 5, Framer Motion 12, React Router 7, Supabase (Postgres + PostGIS + Auth + Realtime).

### Routing (`App.tsx`)

```
/                   → redirects to /discover
/discover           → DiscoverPage (map + pin card popover on desktop, bottom sheet feed on mobile)
/post               → PostPage (creation method: Pin / Establishment / Transit)
/post/pin           → PostPinPage (pin-on-map + post form)
/post/establishment → PostEstablishmentPage (place search + post form)
/messages           → MessagesPage (conversation list)
/messages/:id       → ChatPage (threaded chat)
/profile            → ProfilePage (placeholder)
```

### State (Zustand stores in `src/stores/`)

Six stores keep state separate from components (designed to be React Native portable):

- **`auth.ts`** — Supabase Auth (OTP email login), `user`, `session`, `profile`, `pendingAction`, `isAuthSheetOpen`. Uses immer middleware. Initializes on app mount via `App.tsx`.
- **`navigation.ts`** — `activeTab`, `createPostMode`
- **`map.ts`** — `selectedDate` (nullable, null = all dates), `genderFilter`, `selectedPost`, `activePostIndex`, `showFeedPanel`, `transitMode`, `viewTransit`
- **`postCreation.ts`** — `droppedPin`, `selectedPlace`, `content`, `seeking`, verification Q&A
- **`conversations.ts`** — Message threads and conversation state
- **`search.ts`** — `query`, `results` (null = inactive, [] = no matches), `isSearching`, `error`

### Layout

`AppShell` wraps all routes in a flex container with `TabBar` always rendered at the bottom. The tab bar reads and writes `activeTab` from the navigation store and syncs with React Router location.

### Discover Page (Desktop vs Mobile)

**Desktop (>=1024px):** Map-first layout — the map fills the full viewport with no sidebar. Clicking a pin opens a `MapPostCard` popover fixed at bottom-center of the map (Apple Maps style). Arrow keys cycle through posts (wrapping), map pans to follow. Escape or click-outside dismisses. A floating `FilterDropdown` sits at top-right of the map. Active pin scales up with a drop shadow. The `DesktopTopBar` has a live `SearchBar` component. Components: `MapPostCard` (card popover), `FilterDropdown` (floating filter), `SearchBar` (unified search input).

**Mobile (<1024px):** `PostRail` slides up as a bottom sheet (`max-h-[60vh]`, rounded top) toggled by a `FeedToggleButton` on the map. Contains a scrollable card list (`FeedPostCard` items) with sort toggle. Selected post detail + "Was This You?" button appears at bottom of the sheet. `FilterDropdown` lives in the mobile brand header. A search icon in the header expands to a `SearchBar` via Framer Motion slide-in; Cancel collapses back to the wordmark.

The `useIsDesktop` hook (`src/hooks/useMediaQuery.ts`, 1024px breakpoint) gates which experience renders. Both read from the same Zustand `map` store, so resizing across the breakpoint is seamless.

### Data

Frontend data is currently hardcoded in `src/lib/constants.ts` — 145 sample posts across dates and transit modes, 2 sample conversations, subway line/color mappings, MTA colors, borough lists, rail lines. Types live in `src/lib/types.ts`. The same 145 posts are seeded into Supabase with pgvector embeddings (see `supabase/seed.sql`). Deterministic UUIDs map between client numeric IDs and Supabase UUIDs: post ID `N` → UUID `10000000-0000-0000-0000-{N zero-padded to 12}`.

### Backend (Supabase)

**Project ref**: `btjcqshfvnzmlcylhzsd` — hosted on Supabase (US East).

**Database schema** (6 tables, all with RLS):
- `users` — profiles, seeking preference, rate limiting, subscription tier
- `posts` — content, PostGIS `geography(Point, 4326)` for location, tsvector for full-text search, `embedding vector(512)` for pgvector semantic search, auto-expiry trigger (3 days), transit metadata
- `verification_attempts` — one attempt per user per post (unique constraint)
- `conversations` — links two participants after verification passes
- `messages` — text/image/contact_share, read receipts
- `contact_channels` — instagram/phone/snapchat/email (Phase 3)

**Key features**: PostGIS geospatial indexes (GIST), full-text search (GIN on tsvector), pgvector HNSW index for semantic search (cosine distance), hybrid `search_posts` RPC (70% vector similarity + 30% tsvector keyword relevance), `public_posts` view that hides `verification_answer_hash` and `embedding` from clients.

**Supabase CLI commands** (run from `app/`):
```bash
supabase db push           # Push migrations to remote
supabase db pull           # Pull remote schema changes
supabase gen types typescript --project-id btjcqshfvnzmlcylhzsd > src/lib/database.types.ts
supabase functions deploy embed-text      # Deploy embedding edge function
supabase functions deploy search-posts    # Deploy search edge function
supabase functions deploy verify-answer   # Deploy verification edge function
supabase secrets set OPENAI_API_KEY=sk-…  # Set OpenAI key for edge functions
```

**Files** (all paths relative to `app/`):
- `supabase/migrations/` — 9 migration files (extensions, tables, RLS, pgvector search)
- `supabase/functions/embed-text/` — Edge Function: embeds text via OpenAI `text-embedding-3-small` (512 dims)
- `supabase/functions/search-posts/` — Edge Function: embeds query + calls `search_posts` RPC for hybrid ranked results
- `supabase/functions/verify-answer/` — Edge Function: handles verification attempt submission, creates conversations on match
- `supabase/seed.sql` — Idempotent seed of 145 sample posts with PostGIS points and deterministic UUIDs
- `src/lib/supabase.ts` — typed Supabase client
- `src/lib/database.types.ts` — Row/Insert/Update types for all tables

### Deployment

Hosting target is **Vercel** (free tier). Config in `app/vercel.json` — SPA rewrites for React Router. Root directory is set to `app/` in the Vercel dashboard. Connect via Vercel + Supabase marketplace integration for automatic env var syncing.

### Prototype (`old/LFMJesse.html`)

Monolithic `LookForMeApp` component using `React.createElement` aliased as `h`. All state in `useState` hooks. Google Maps with custom markers and clustering (`clusterPosts()` groups posts within ~0.01 degrees). Keep `old/LFMLastGoodOne.html` as a rollback point — copy the working file there before major edits.

## Design Constraints

- **iPhone 16 viewport** (393px width) — `#root` is max-width constrained with `safe-area-inset-bottom`
- **Color palette**: dark teal `#2f4f4f`, accent pink `#fc9dce`, male-seeking blue `#7cb6f7`
- **Font**: Plus Jakarta Sans (brand/wordmark), system-ui fallback for body
- **MTA subway colors** are defined in constants for transit views

## CSS / Tailwind v4 Gotcha

**Never add unlayered CSS resets in `index.css`.** Tailwind v4 puts all utilities inside CSS cascade layers (`@layer base`, `@layer utilities`). Per the CSS spec, unlayered styles always beat layered styles regardless of specificity. An unlayered `* { padding: 0 }` will silently kill every Tailwind spacing class in the entire app.

All custom styles in `index.css` must go inside `@layer base` (resets, element defaults) or `@layer utilities` (custom utility classes like `.shadow-warm-*`). Tailwind v4's preflight already handles `box-sizing: border-box` and margin/padding resets within its layer system — do not duplicate them.

## Known Issues

- **Post > Transit flow** — Bottom nav bar disappears (blocked development on the prototype)
- **Pin on Map** — Map pin interaction is incomplete
- **Search Establishment** — Google Places integration not implemented
- **Discover search** — Live hybrid search (pgvector + tsvector) via Supabase edge functions; results depend on seeded embeddings

## Roadmap

### Unified Search (pgvector hybrid)

**Status**: Fully wired end-to-end.

Typing in the search bar (desktop `DesktopTopBar`, mobile expandable header) calls the `search-posts` edge function after a 400ms debounce. The edge function embeds the query via OpenAI `text-embedding-3-small` (512 dims), then calls the `search_posts` Postgres RPC which combines pgvector cosine similarity (70%) with tsvector keyword relevance (30%). Results filter map pins (non-matching pins dim to 25% opacity + grayscale) and populate the feed panel. On mobile, the PostRail auto-opens when results arrive.

**Key files**:
- `src/stores/search.ts` — Zustand store (`query`, `results`, `isSearching`, `error`, `clearSearch`)
- `src/hooks/useSearch.ts` — 400ms debounced effect calling `search-posts` edge function via `supabase.functions.invoke`
- `src/components/discover/SearchBar.tsx` — Pill-shaped input with spinner, clear button, Escape key
- `supabase/functions/search-posts/index.ts` — Edge Function: embed query → call `search_posts` RPC
- `supabase/functions/embed-text/index.ts` — Edge Function: standalone text → embedding
- `supabase/migrations/20250201000009_add_pgvector_search.sql` — pgvector extension, `embedding` column, HNSW index, `search_posts` RPC

**Seeding embeddings**: `scripts/seed-embeddings.ts` (Deno) backfills embeddings for posts where `embedding IS NULL`. Run after `seed.sql` populates posts. Posts get deterministic UUIDs: `10000000-0000-0000-0000-{id padded to 12}`.

**What's next**:
- Embed posts at creation time (call `embed-text` from the post creation flow or a DB trigger)
- Replace SAMPLE_POSTS → Supabase query so search results fetch live data instead of mapping UUIDs back to hardcoded array
- Add location-biased search (boost results near the current map viewport)

### Auth (Supabase OTP)

**Status**: Implemented (store + UI + edge function).

Email OTP magic-link flow via Supabase Auth. The `auth.ts` store initializes on mount, listens for `onAuthStateChange`, and auto-creates a `users` row on first sign-in. `AuthSheet` component (bottom sheet on mobile, modal on desktop) gates protected actions via `pendingAction` — when an unauthenticated user tries to verify a post or create one, the auth sheet opens first, then resumes the action after sign-in.

**Key files**: `src/stores/auth.ts`, `src/components/auth/AuthSheet.tsx`, `authplan.md` (implementation plan)

### Verification Flow (soft-signal model)

**Status**: Frontend + backend implemented.

"Was This You?" opens a `VerificationSheet` — bottom sheet on mobile, centered modal on desktop. The poster sets a verification question; the responder answers it. All responses are forwarded to the poster (no bcrypt pass/fail gate). The poster decides who to engage with.

**Key files**:
- `src/components/discover/VerificationSheet.tsx` — two-step UI (form → success), drag-to-dismiss, Escape on desktop
- `src/lib/verification.ts` — `submitVerification()` (currently mocked with 600ms delay, to be wired to Supabase)
- `supabase/functions/verify-answer/index.ts` — Deno Edge Function: inserts `verification_attempts` row + creates `conversations` row
- 15 sample posts have `verificationQuestion` strings; posts without one get a fallback prompt

**What's next**:
- Wire `src/lib/verification.ts` to call the deployed `verify-answer` Edge Function instead of the mock
- Push notification to poster when a new verification response arrives

## Testing

Uses **Vitest 4**. Test files live in `__tests__/` directories alongside their source:

- `src/stores/__tests__/auth.test.ts` — Auth store tests
- `src/stores/__tests__/conversations.test.ts` — Conversations store tests
- `src/lib/__tests__/verification.test.ts` — Verification logic tests

```bash
cd app
npm run test              # Run all tests once
npx vitest                # Watch mode
npx vitest run auth       # Run tests matching "auth"
```

## Key Files

- **`old/spec.md`** — Full product and technical specification (features, phases, data model, design philosophy)
- **`old/convo.md`** — Creator's original context and requirements
- **`authplan.md`** — Supabase Auth implementation plan (4 phases A–D)
- **`.playwright-mcp/`** — Reference screenshots of each tab/view (discover, post, transit, messages, chat)
- **`scripts/seed-embeddings.ts`** — Deno script to backfill pgvector embeddings via OpenAI API

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Uchu1222) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
