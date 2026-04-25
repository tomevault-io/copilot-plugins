## goalbet

> Authoritative reference for Claude when working in this repository.

# GoalBet — CLAUDE.md

Authoritative reference for Claude when working in this repository.
Read this before touching any file. Everything here reflects the live codebase.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Commands](#3-commands)
4. [Critical Rules](#4-critical-rules)
5. [File & Folder Map](#5-file--folder-map)
6. [Routing](#6-routing)
7. [Architecture Overview](#7-architecture-overview)
8. [Auth System](#8-auth-system)
9. [Sync System](#9-sync-system)
10. [Scoring System](#10-scoring-system)
11. [Coin Economy](#11-coin-economy)
12. [Match Status System](#12-match-status-system)
13. [League System & ESPN Coverage](#13-league-system--espn-coverage)
14. [Database & Migrations](#14-database--migrations)
15. [Theme System](#15-theme-system)
16. [Internationalisation (i18n)](#16-internationalisation-i18n)
17. [Stores](#17-stores)
18. [Hooks](#18-hooks)
19. [CI / GitHub Actions](#19-ci--github-actions)
20. [Admin Console](#20-admin-console)
21. [Common Pitfalls](#21-common-pitfalls)

---

## 1. Project Overview

**GoalBet** is a non-commercial football prediction game for friend groups.
Users sign up with email/password or Google, join private groups via invite code, predict match outcomes across 5 tiers, stake coins, and compete on a real-time leaderboard.

**Live URL:** Auto-deploys from `main` via Vercel.
**Supabase project:** `rzavwyejcldvztkykhks`
**Backend:** Hosted on Render (free tier — sleeps after ~15 min inactivity).
**Primary market:** Israel (Hebrew-first, RTL support, midnight Israel timezone for daily bonuses).
**Zero paid APIs:** ESPN public scoreboard (no key), Supabase free tier, Render free tier.

---

## 2. Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Build | Vite | ^5.0 |
| UI | React | ^18.2 |
| Language | TypeScript | ^5.3 (strict) |
| Styling | Tailwind CSS | ^3.4 |
| Animation | Framer Motion | ^12.36 |
| Routing | react-router-dom | ^6.21 |
| State | Zustand | ^4.5 |
| Backend/DB | Supabase JS | ^2.39 |
| Icons | Lucide React | ^0.577 |
| Backend runtime | Node.js + Express | ^4.18 |
| Backend cron | node-cron | ^3.0 |
| Backend HTTP client | axios | ^1.6 |

**No** Redux, MUI, Chakra, styled-components, class-based components, or TheSportsDB (replaced by ESPN).

---

## 3. Commands

```bash
# Frontend — http://localhost:5173
cd frontend && npm run dev

# Backend — http://localhost:3001
cd backend && npm run dev

# Type-check (run before committing)
cd frontend && npx tsc --noEmit

# Build (what CI runs — must pass)
cd frontend && npm run build

# Manual match sync (dev only)
cd backend && npm run sync

# Seed dev data
cd backend && npm run seed

# Deploy all pending migrations to remote Supabase
supabase db push --linked

# Check migration status (what's applied vs pending)
supabase migration list --linked
```

**Required env files** — never committed to git:

```
frontend/.env.local
  VITE_SUPABASE_URL=https://rzavwyejcldvztkykhks.supabase.co
  VITE_SUPABASE_ANON_KEY=<anon key>
  VITE_BACKEND_URL=http://localhost:3001

backend/.env
  SUPABASE_URL=https://rzavwyejcldvztkykhks.supabase.co
  SUPABASE_SERVICE_ROLE_KEY=<service role key>
  PORT=3001
  NODE_ENV=development
```

No feature flags required. Email + password auth is active by default; no env var needed.

### Auto-migration hook

`.claude/settings.local.json` contains a PostToolUse hook that automatically runs
`supabase db push --linked` whenever Claude writes or edits a file matching
`supabase/migrations/*.sql`. This requires the Supabase CLI to be authenticated:

```bash
supabase login   # one-time browser auth — do this before working on migrations
```

After login, every migration file Claude writes is auto-deployed to the remote DB.

---

## 4. Critical Rules

These have caused build failures or subtle bugs in the past. Never violate them.

### 4.1 Framer Motion `ease` type

```tsx
// ❌ Breaks tsc — number[] is not assignable to Easing in some contexts
transition={{ ease: [0.4, 0, 1, 1] }}

// ✅ Correct
transition={{ ease: 'easeIn' as const }}
transition={{ ease: 'easeOut' as const }}
transition={{ ease: 'easeInOut' as const }}
```

### 4.2 Framer Motion `Variants` — import the type

```tsx
import { type Variants } from 'framer-motion'

const slideVariants: Variants = {
  enter: (dir: number) => ({ x: dir > 0 ? 28 : -28, opacity: 0 }),
  center: { x: 0, opacity: 1 },
  exit:   (dir: number) => ({ x: dir > 0 ? -28 : 28, opacity: 0,
    transition: { duration: 0.14, ease: 'easeIn' as const } }),
}
```

### 4.3 AppShell owns ALL auto-sync — never add sync to page components

```tsx
// ❌ Creates race conditions and infinite spinners
useEffect(() => { fetch('/api/sync/matches') }, [])  // inside a page component

// ✅ AppShell.tsx is the only file that triggers automatic sync
// Pages dispatch 'goalbet:synced' listeners — never the triggerers
```

### 4.4 Realtime UPDATE events — always re-fetch, never swap in-place

```tsx
// ❌ The Realtime payload is PARTIAL — JSONB columns (predicted_*, add_ons) are missing
.on('postgres_changes', { event: 'UPDATE', ... }, (payload) => {
  setMatches(prev => prev.map(m => m.id === payload.new.id ? payload.new : m))
})

// ✅ Always call a full re-fetch
.on('postgres_changes', { event: 'UPDATE', ... }, () => {
  fetchMatches()
})
```

### 4.5 Prediction lock must be validated server-side

Kickoff time is checked client-side for UX, but the Supabase RLS/trigger also validates it. Never rely only on the client-side `kickoff_time < now()` check for security.

### 4.6 Never use `CURRENT_DATE` for daily bonus — use Israel timezone

```sql
-- ❌ Uses UTC, resets at wrong time for Israeli users
WHERE claim_date = CURRENT_DATE

-- ✅ Israel timezone (Asia/Jerusalem)
WHERE claim_date = (NOW() AT TIME ZONE 'Asia/Jerusalem')::DATE
```

### 4.7 Scoring uses regulation time, not final score

For knockout matches that go to extra time or penalties, scoring always uses `regulation_home` / `regulation_away`, not `home_score` / `away_score`.

```typescript
// In pointsEngine.ts and calcBreakdown() — always use:
const homeGoals = match.regulation_home ?? match.home_score
const awayGoals = match.regulation_away ?? match.away_score
```

### 4.8 Two coexisting Tier 3 fields — never remove either

Old predictions: `predicted_halftime_outcome` (half-time result, pre-migration 014).
New predictions: `predicted_corners` (corners, post-migration 014).
Both fields exist on the `predictions` table. `calcBreakdown()` handles both. Never remove either.

### 4.9 `Avatar` expects `emoji:🏆` prefix format

```tsx
// ❌ Treated as image URL, silently fails
<Avatar src="🏆" />

// ✅ Correct — prefix tells Avatar it's an emoji
<Avatar src="emoji:🏆" />
```

### 4.10 Use logical CSS properties for RTL compatibility

```tsx
// ❌ Breaks Hebrew RTL layout
className="ml-2 pl-4"

// ✅ Logical properties flip automatically in RTL
className="ms-2 ps-4"
```

### 4.11 Admin RPCs are SECURITY DEFINER — never skip the email check

```sql
-- ❌ Skipping the admin guard exposes destructive RPCs to any authenticated user
CREATE OR REPLACE FUNCTION admin_delete_group(p_group_id UUID) ...

-- ✅ Every admin function must call is_super_admin() as its FIRST action
BEGIN
  IF NOT is_super_admin() THEN
    RAISE EXCEPTION 'Access denied: super-admin only';
  END IF;
  ...
```

### 4.12 Coins live in `group_members.coins` — not a separate column

### 4.13 All bottom-sheet modals must implement swipe-to-close

Every modal that slides up from the bottom on mobile uses the Framer Motion drag pattern. Apply these props to the outer panel `motion.div`:

```tsx
drag="y"
dragConstraints={{ top: 0 }}
dragElastic={0.15}
dragMomentum={false}
onDragEnd={(_, info) => {
  if (info.offset.y > 100 && info.velocity.y > 20) onClose();
}}
```

And on every **scroll container inside** that modal:

```tsx
onPointerDown={e => e.stopPropagation()}
```

Without `onPointerDown` on scroll containers, Framer Motion intercepts touch events and the user can't scroll — the drag fires instead.

Modals with swipe-to-close: `HelpGuideModal`, `ScoringGuide`, `CoinGuide`, `CoinHistoryModal`, `UserMatchHistoryModal`.

### 4.14 Score resolution must use the atomic claim pattern + matchEndAt timestamps

Concurrent score resolvers (GitHub Actions cron every 5 min + Render scheduler every 30 s + manual sync from the Settings page) all SELECT the same `is_resolved=false` predictions. Without a guard they all proceed, all award coins, all insert notifications and locker-room events. This caused real duplicates in production (cleaned up by migrations 031 + 033).

**Every prediction-resolving code path in `scoreUpdater.ts` must:**

```typescript
// 1. Atomic claim — only one concurrent worker wins
const { data: claimed } = await supabaseAdmin
  .from('predictions')
  .update({ points_earned: finalPoints, is_resolved: true })
  .eq('id', prediction.id)
  .eq('is_resolved', false)        // ← guard
  .select('id');

if (!claimed || claimed.length === 0) continue;  // we lost — skip side effects

// 2. matchEndAt for every derived row (coin txn, notification, group_event)
const matchEndAt = new Date(new Date(kickoff_time).getTime() + 105 * 60 * 1000).toISOString();
```

The corners re-score loop uses a different claim shape because `is_resolved` is already true: `.lt('points_earned', correctBreakdown.total)` so only the first writer with the new (higher) total wins.

The DB has three partial unique indexes as a final backstop — see rule 4.15.

### 4.15 Never drop the coin/event/notification dedup constraints

Three partial unique indexes prevent duplicate awards even if application logic regresses. Do not drop these without a written, reviewed reason:

- `coin_transactions_bet_won_unique` — `(user_id, group_id, match_id, description) WHERE type='bet_won' AND match_id IS NOT NULL` (migration 032)
- `group_events_won_coins_unique` — `(group_id, user_id, match_id) WHERE event_type='WON_COINS' AND match_id IS NOT NULL` (migration 033)
- `notifications_prediction_result_unique` — `(user_id, (metadata->>'match_id')) WHERE type='prediction_result' AND metadata ? 'match_id'` (migration 033)

`description` is intentionally part of the coin key — the original "Won X pts → Y coins" award and a later "Corners re-score: +Z pts → +W coins" top-up are legitimate distinct rows for the same match. Removing `description` from the index would silently swallow the corners top-up.

`increment_coins` (after migration 032) relies on `coin_transactions_bet_won_unique` as its `ON CONFLICT` target. If the index is dropped, the function's `ON CONFLICT (user_id, group_id, match_id, description) WHERE …` clause throws at runtime — coin awards stop entirely. Recreate the index immediately if it ever goes missing.

### 4.16 Always set explicit `width` and `height` on `<img>` tags

All `<img>` elements must have numeric `width` and `height` HTML attributes so the browser reserves layout space before the image loads. Omitting them causes Cumulative Layout Shift (CLS) on slow connections.

```tsx
// ❌ CLS — browser doesn't know the size until the image loads
<img src={badge} alt={team} className="w-9 h-9" />

// ✅ Browser reserves 36×36px immediately
<img src={badge} alt={team} width={36} height={36} className="w-9 h-9" />
```

This applies to team badges, league logos, and avatars loaded from remote URLs.

The authoritative coin balance for a user in a group is `group_members.coins`.
`user_coins` is a legacy/supplementary table (created in migration 020) but the
primary balance used by `increment_coins`, `claim_daily_bonus`, and all admin RPCs
is `group_members.coins`.

```sql
-- ✅ Correct — read/write coins from group_members
UPDATE group_members SET coins = GREATEST(0, coins + p_delta)
WHERE user_id = p_user_id AND group_id = p_group_id;

-- ❌ Do not read totals from user_coins for balance display
```

---

## 5. File & Folder Map

```
goalbet/
├── frontend/
│   └── src/
│       ├── components/
│       │   ├── admin/
│       │   │   ├── AdminLayout.tsx        # Admin shell: sidebar (desktop) + top bar (mobile), Outlet
│       │   │   ├── AdminProtectedRoute.tsx # Email guard: only roychen651@gmail.com; silently redirects
│       │   │   ├── DangerModal.tsx        # Confirms destructive actions — requires typing "DELETE"
│       │   │   ├── GroupManagement.tsx    # Rename, view members, delete groups with DangerModal
│       │   │   └── UserManagement.tsx     # Edit name, manage coins per group, reset password, delete
│       │   ├── auth-v2/
│       │   │   ├── AuthContainer.tsx      # Master auth UI — 8-view glassmorphism card
│       │   │   ├── PasswordStrength.tsx   # Animated strength meter + 5 SVG checkmarks
│       │   │   └── ReAuthModal.tsx        # Session-expiry overlay — slides in over current page
│       │   ├── auth/
│       │   │   └── GoogleLoginButton.tsx  # Legacy Google button (unused in auth-v2 flow)
│       │   ├── groups/
│       │   │   ├── CreateGroupModal.tsx
│       │   │   ├── InviteCodeDisplay.tsx
│       │   │   └── JoinGroupModal.tsx
│       │   ├── layout/
│       │   │   ├── AppShell.tsx           # Root layout + ONLY place for auto-sync + cold-start isSyncing flag
│       │   │   ├── BottomNav.tsx          # Mobile bottom navigation
│       │   │   ├── ErrorBoundary.tsx      # Class component — catches render errors, shows bilingual fallback
│       │   │   ├── Sidebar.tsx            # Desktop sidebar
│       │   │   └── TopBar.tsx             # Mobile header: logo, group selector, coins, avatar
│       │   ├── leaderboard/
│       │   │   ├── H2HModal.tsx           # Head-to-head comparison (tap another user's row)
│       │   │   ├── LeaderboardRow.tsx     # Own row → history modal; other row → H2H modal
│       │   │   ├── LeaderboardTable.tsx
│       │   │   └── UserMatchHistoryModal.tsx  # Bottom sheet — swipe-to-close enabled
│       │   ├── matches/
│       │   │   ├── MatchCard.tsx          # ACTIVE card: MatchCardCore (private) + MatchCard (public, shimmer wrapper). isPastKickoffNS, DELAYED, live clock, weather/referee/competition phase, TacticalIntelSection, dual dark/light league logos, live breathing glow, goal flash, score flip
│       │   │   ├── MatchFeed.tsx          # Date-grouped feed; imports MatchCard directly
│       │   │   ├── MatchRosters.tsx       # Starting XI + substitutes fetched from ESPN; feeds TacticalPitch
│       │   │   ├── MatchStats.tsx         # Post-match statistics (possession, shots, corners, etc.)
│       │   │   ├── MatchStatusBadge.tsx   # Status pill; intercepts DELAYED→SYNCING during cold-start
│       │   │   ├── MatchTimeline.tsx      # ESPN summary events (returns null when no data)
│       │   │   ├── TacticalPitch.tsx      # Glass tactical formation view for Starting XI; horizontal pitch with percentage-based positioning
│       │   │   └── PredictionForm.tsx     # 5-tier prediction input; corners hidden for league 4396
│       │   └── ui/
│       │       ├── Avatar.tsx             # Expects emoji:🏆 prefix
│       │       ├── CoinGuide.tsx          # Bottom sheet — swipe-to-close enabled
│       │       ├── CoinHistoryModal.tsx   # Bottom sheet — swipe-to-close enabled
│       │       ├── CoinIcon.tsx           # Animated coin SVG icon with configurable size
│       │       ├── EmptyState.tsx         # Reusable empty-state placeholder
│       │       ├── FadeInView.tsx         # Wrapper: fade-in on mount via Framer Motion
│       │       ├── GlassCard.tsx
│       │       ├── HelpGuideModal.tsx     # Bottom sheet — swipe-to-close enabled
│       │       ├── InfoTip.tsx            # Tooltip using CSS vars (works in both themes)
│       │       ├── LangToggle.tsx
│       │       ├── LoadingSpinner.tsx
│       │       ├── MagneticButtonV2.tsx   # Magnetic pull button; variants: volt / ghost / purple
│       │       ├── NeonButton.tsx         # Variants: green / ghost / danger
│       │       ├── PolicyModal.tsx
│       │       ├── ScoringGuide.tsx       # Bottom sheet — swipe-to-close enabled
│       │       ├── StaggerList.tsx        # Wrapper: staggered child animations
│       │       ├── SyncProgressBar.tsx    # Fixed top bar; visible while isSyncing; z-[100]
│       │       ├── ThemeToggle.tsx
│       │       ├── TiltCardV2.tsx         # 3° tilt with spring physics for profile bento cards
│       │       ├── Toast.tsx
│       │       └── WelcomeAnimation.tsx   # First-login welcome sequence
│       ├── hooks/
│       │   ├── useAuth.ts                 # Legacy Google OAuth (kept for backward compat)
│       │   ├── useAuthV2.ts               # Auth-v2 state machine (8 views)
│       │   ├── useGroupMatchPredictions.ts
│       │   ├── useLeaderboard.ts
│       │   ├── useLiveClock.ts            # Ticking clock for live matches
│       │   ├── useMatches.ts              # Fetches + Realtime + goalbet:synced listener
│       │   ├── useMatchSync.ts            # Manual sync ONLY (Settings button) — 60s timeout
│       │   ├── useNewPointsAlert.ts       # Toast on newly earned points since last visit
│       │   ├── usePredictions.ts
│       │   └── useRTLDirection.ts         # Sets document.dir from active language
│       ├── lib/
│       │   ├── authSchema.ts              # Password validation: strength, requirements, error mapping
│       │   ├── constants.ts               # FOOTBALL_LEAGUES, LEAGUE_ESPN_SLUG, POINTS, COIN_COSTS, ROUTES
│       │   ├── featureFlags.ts            # Feature flag registry (currently no active flags)
│       │   ├── i18n.ts                    # EN + HE translations, TranslationKey type
│       │   ├── supabase.ts                # Supabase client (anon key) + all TypeScript table types
│       │   └── utils.ts                   # calcBreakdown() (client-side scoring mirror), cn()
│       ├── pages/
│       │   ├── admin/
│       │   │   └── AdminDashboardPage.tsx # Bento grid KPIs + system health actions
│       │   ├── AuthCallbackPage.tsx       # Handles Supabase OAuth redirect
│       │   ├── HomePage.tsx               # Match feed — All / Upcoming / Live / Results tabs
│       │   ├── LeaderboardPage.tsx        # Group standings with H2H modal
│       │   ├── LoginPage.tsx              # Thin wrapper — redirect if logged in, render AuthContainer
│       │   ├── ProfilePage.tsx            # Stats, prediction history, sign-out button
│       │   └── SettingsPage.tsx           # Group mgmt, leagues, admin tools, Account section
│       └── stores/
│           ├── authStore.ts               # user, profile, session; signInWithGoogle, signOut
│           ├── coinsStore.ts              # coins, lastDailyBonus; synced from DB
│           ├── groupStore.ts              # groups[], activeGroupId; persisted to localStorage
│           ├── langStore.ts               # lang ('en'|'he'); persisted to localStorage
│           ├── themeStore.ts              # theme ('dark'|'light'); persisted to localStorage
│           └── uiStore.ts                 # activeModal, toasts[], isSyncing; memory only
│
├── backend/
│   └── src/
│       ├── cron/
│       │   └── scheduler.ts               # Startup catch-up + 30s score poller + daily/weekly crons
│       ├── lib/
│       │   └── supabaseAdmin.ts           # Supabase client with service-role key (bypasses RLS)
│       ├── routes/
│       │   ├── admin.ts                   # DELETE /api/admin/users/:id · POST /api/admin/reset-password
│       │   ├── health.ts                  # GET /api/health → { status: 'ok' }
│       │   └── sync.ts                    # POST /api/sync/matches · POST /api/sync/scores
│       ├── scripts/
│       │   ├── manualSync.ts              # npm run sync — dev helper
│       │   └── seed.ts                    # npm run seed — populates dev data
│       └── services/
│           ├── espn.ts                    # ESPN API client + LEAGUE_ESPN_MAP
│           ├── matchSync.ts               # syncLeague(id), syncAllActiveLeagues()
│           ├── pointsEngine.ts            # PURE scoring function — no DB calls, fully testable
│           ├── scoreUpdater.ts            # Resolves predictions after FT, writes leaderboard + coins
│           └── sportsdb.ts               # DBMatch type definition (legacy, kept for types)
│
├── supabase/
│   ├── email-templates/
│   │   ├── confirm-signup.html            # Green-theme onboarding email (paste into Supabase dashboard)
│   │   └── reset-password.html            # Orange-theme recovery email (paste into Supabase dashboard)
│   └── migrations/                        # 001 → 023 — auto-deployed via PostToolUse hook on write
│
├── .claude/
│   └── settings.local.json                # PostToolUse hook: auto-runs `supabase db push --linked` on migration writes
│
└── .github/
    └── workflows/
        ├── ci.yml                         # Type-check + build on every push/PR
        └── sync-cron.yml                  # Every 5 min: wake → scores (critical) → fixtures (non-critical)
```

---

## 6. Routing

```
/login           → LoginPage          (public — renders AuthContainer)
/auth/callback   → AuthCallbackPage   (public — Supabase Google OAuth redirect)
/                → HomePage           (protected via AuthGuard → AppShell)
/leaderboard     → LeaderboardPage    (protected via AuthGuard → AppShell)
/profile         → ProfilePage        (protected via AuthGuard → AppShell)
/settings        → SettingsPage       (protected via AuthGuard → AppShell)
/admin           → AdminDashboardPage (protected via AdminProtectedRoute → AdminLayout)
/admin/users     → UserManagement     (protected via AdminProtectedRoute → AdminLayout)
/admin/groups    → GroupManagement    (protected via AdminProtectedRoute → AdminLayout)
*                → redirect to /
```

**`AuthGuard`** (in `App.tsx`):
- Renders `<PageLoader />` while `!initialized || loading`
- On unexpected session expiry (user was logged in → now null): shows `<ReAuthModal>` instead of redirecting
- On deliberate sign-out with no session: `<Navigate to="/login" replace />`

**`AdminProtectedRoute`** (in `components/admin/AdminProtectedRoute.tsx`):
- Client-side email check: `user.email === 'roychen651@gmail.com'`
- Silently redirects to `/` for any other user — no error message shown
- Server-side: every admin RPC calls `is_super_admin()` as its first action (double protection)

**`AppInitializer`** (wraps entire app):
- Calls `authStore.init()` once on mount (restores session, subscribes to `onAuthStateChange`)
- Calls `groupStore.fetchGroups(user.id)` on user change
- Calls `coinsStore.initCoins(user.id, activeGroupId)` on user/group change + `visibilitychange` for midnight Israel timezone reset

---

## 7. Architecture Overview

```
Browser
  │
  ├─ Vercel (React SPA — auto-deploy from main)
  │    ├─ AppShell         → owns ALL automatic sync; manages isSyncing cold-start flag
  │    ├─ SyncProgressBar  → fixed top shimmer; visible while isSyncing=true
  │    ├─ ErrorBoundary    → wraps <Outlet />; catches render errors; bilingual fallback
  │    ├─ useMatches       → queries Supabase + listens for 'goalbet:synced' event + Realtime
  │    └─ useMatchSync     → Settings "Sync Now" button ONLY (manual, never auto-triggered)
  │
  ├─ Supabase Realtime     → pushes match UPDATE/INSERT to subscribed clients
  │
  └─ Render (Express API — free tier, sleeps after ~15 min)
       ├─ POST /api/sync/matches  → syncAllActiveLeagues() pulls fixtures from ESPN
       ├─ POST /api/sync/scores   → checkAndUpdateScores() resolves finished predictions + awards coins
       ├─ GET  /api/health        → { status: 'ok' }
       ├─ DELETE /api/admin/users/:id     → hard-delete from auth.users (service role, admin only)
       ├─ POST  /api/admin/reset-password → send password reset email (service role, admin only)
       └─ setInterval 30s         → live score polling while awake
```

**Data flow on page load:**
1. `AppShell` sets `isSyncing = true` → `SyncProgressBar` appears + DELAYED badges show "Syncing…"
2. `AppShell` fires `POST /api/sync/matches` (90s timeout) — wakes Render + pulls fresh fixtures
3. `AppShell` fires `POST /api/sync/scores` immediately (75s timeout) + retries at 25s
4. First successful response: `dispatchAndClearSyncing()` → `isSyncing = false` + `goalbet:synced`
5. `useMatches` hears `goalbet:synced` → calls `fetchMatches()` → UI updates
6. Supabase Realtime also pushes row-level changes for live score diffs

**GitHub Actions as heartbeat:**
`sync-cron.yml` runs every **5 minutes**. Wakes backend → resolves scores (coins) → syncs fixtures. Keeps Render alive 24/7 and data current even with zero active users.

---

## 8. Auth System

### Sign-in methods

| Method | Flow |
|--------|------|
| Google OAuth | `authStore.signInWithGoogle()` → Supabase OAuth redirect → `/auth/callback` → `authStore.onAuthStateChange` handles session → `LoginPage.useEffect` redirects to `/` |
| Email + password | `AuthContainer` → `useAuthV2` → `supabase.auth.signInWithPassword()` |
| Sign up | `AuthContainer` sign-up view → `supabase.auth.signUp()` with `data.username` → confirm email → back to sign-in |
| Forgot password | `AuthContainer` forgot view → `supabase.auth.resetPasswordForEmail()` redirecting to `/login?type=recovery` |
| Password recovery | `useAuthV2` detects `?type=recovery` on mount → verifies session → navigates to `set-password` view → `supabase.auth.updateUser({ password })` |
| Change password (in-app) | `SettingsPage` Account section → `supabase.auth.updateUser({ password })` directly |

### `useAuthV2` — view state machine

```
email ──continue──► signin ──success──► success (→ redirect)
  │                   │
  │              wrong creds: show error + Google hint
  │
  └──create account──► signup ──already registered──► oauth-merge
                         │
                         └──success──► check-email (signup context)

email ──forgot──► forgot ──always──► check-email (reset context)

/login?type=recovery ──on mount──► set-password ──success──► success
```

**Identity collision detection:**
`supabase.signUp()` returning "User already registered" (HTTP 422) is the only reliable client-side signal that an email belongs to a Google-only account. UI morphs to `oauth-merge` view. Never use a separate "check email" endpoint — that breaks anti-enumeration.

**Anti-enumeration on forgot password:**
`handleForgotPassword()` swallows all errors and always navigates to `check-email`. Never show "email not found" — that reveals account existence.

**Password recovery URL:**
`resetPasswordForEmail` redirects to `/login?type=recovery` (not `/auth/callback`). `useAuthV2` detects this on mount, verifies the recovery session, navigates to `set-password`, and cleans the URL with `history.replaceState`.

### `authStore` API

```typescript
// Called once on app mount — restores session, subscribes to onAuthStateChange
init(): () => void

// Google OAuth — browser redirects immediately
signInWithGoogle(): Promise<void>

// Signed-in users
signOut(): Promise<void>
fetchProfile(): Promise<void>
updateUsername(username: string): Promise<void>
```

State: `user`, `session`, `profile`, `loading`, `initialized`

### `ReAuthModal`

Appears when session expires mid-session (user was authenticated → now null, without a deliberate sign-out). Shows over the current page without losing any UI state. Has three sub-views: `main` (Google or email/password options), `password` (email + password form), `set-password` (new password + PasswordStrength).

`onSuccess` prop: clears the modal so the user continues seamlessly.
`onSignOut` prop: resets `hadUser` ref, calls `signOut()`, navigates to `/login`.

### `PasswordStrength` component

Used in two places: `AuthContainer` (sign-up view) and `SettingsPage` (change password inline form).

- 4 animated bars with `scaleX` + stagger
- 5 requirement rows (8 chars, uppercase, lowercase, number, special char) with animated SVG path checkmarks
- Color: weak → red, fair → amber, strong → lime, very-strong → accent-green
- `PasswordStrength` must not be duplicated — import from `auth-v2/PasswordStrength`

### `authSchema.ts` — validation functions

```typescript
checkPasswordRequirements(password: string): PasswordRequirements
getPasswordStrength(password: string): PasswordStrength  // 'empty'|'weak'|'fair'|'strong'|'very-strong'
isPasswordValid(password: string): boolean               // all 5 requirements met
isEmailValid(email: string): boolean
isUsernameValid(username: string): boolean               // 2-30 chars, Unicode (Hebrew supported)
mapAuthError(message: string): string                    // maps Supabase error strings → human-readable
```

---

## 9. Sync System

> Read this section before touching any sync-related code.

### Single source of truth: `AppShell.tsx`

`AppShell` is the **only** place that triggers automatic background sync. Do not add auto-sync to any page component — it creates race conditions and infinite spinners.

```
Mount
  ├─ setSyncing(true)        — SyncProgressBar appears; DELAYED badges swap to "Syncing…" blue
  ├─ POST /api/sync/matches  (90s AbortController timeout) — wakes Render + pulls fixtures
  ├─ POST /api/sync/scores   (75s timeout) — resolves any finished predictions immediately
  ├─ setTimeout 25s → POST /api/sync/scores — backend is warm by now, fast retry
  └─ setInterval 30s → POST /api/sync/scores — live-score polling

Tab restore (hidden > 90s) → force POST /api/sync/scores
POLL_THROTTLE_MS = 30,000  → debounces rapid successive calls
```

All fetches use `AbortController` with explicit timeouts. If the backend times out on cold start, the fetch aborts cleanly — `isSyncing` is never permanently stuck because the 25s retry will succeed once Render is warm.

### Cold-start UX: `isSyncing` flag

`AppShell` sets `isSyncing = true` on mount and clears it exactly once — on the first successful sync response — via `dispatchAndClearSyncing()`:

```typescript
const dispatchAndClearSyncing = useCallback(() => {
  if (!hasSyncedRef.current) {
    hasSyncedRef.current = true;
    setSyncing(false);          // clears SyncProgressBar
  }
  dispatch();                   // fires goalbet:synced
}, [setSyncing]);
```

`hasSyncedRef` prevents `setSyncing(false)` from firing more than once per mount, so polling calls don't flicker the progress bar.

**What `isSyncing` controls:**
- `SyncProgressBar` — fixed top shimmer bar visible during cold start
- `MatchStatusBadge` — DELAYED status shows blue "Syncing…" instead of alarming orange "Delayed"

**Why 75s for score sync?** Render free tier takes 45–60s to cold start. The previous 30s timeout meant score sync was always failing on cold start, causing coin payouts to be delayed until the next GitHub Actions run.

**Why 90s tab restore threshold?** Live games update every minute. The previous 5-minute threshold meant users returning to the tab during a live match would see data up to 5 minutes stale before a forced refresh.

### Manual sync: `useMatchSync.ts`

Used **only** by the Settings page "Sync Now" button. 60s timeout. Never add an auto-trigger `useEffect` inside `useMatchSync` — it was removed deliberately.

After completing, dispatches `goalbet:synced` so data hooks refetch.

### Backend startup catch-up: `scheduler.ts`

5 seconds after the Render backend restarts (cold start):
1. `syncAllActiveLeagues()` — pulls any fixtures missed during sleep
2. `checkAndUpdateScores()` — resolves any predictions that finished while backend was down

### `syncAllActiveLeagues()` filtering

```typescript
// Silently skips leagues not in LEAGUE_ESPN_MAP
leagueIds = leagueIds.filter(id => id in LEAGUE_ESPN_MAP)
```

If a league is in `FOOTBALL_LEAGUES` (frontend) but NOT in `LEAGUE_ESPN_MAP` (backend `espn.ts`), it silently never syncs. **Always add to both.**

### Fixture window

`syncLeague()` calls `fetchLeagueMatches(leagueId, 7, 42)` — **7 days back, 42 days ahead**.

### The `goalbet:synced` event

```typescript
window.dispatchEvent(new Event('goalbet:synced'))
```

`useMatches` listens for this and calls `fetchMatches()`. Any component needing a refetch after sync should listen for this event, not poll independently.

---

## 10. Scoring System

| Tier | What is predicted | Points |
|------|-------------------|--------|
| 1 | Full-time result (H / D / A) | **+3** |
| 2 | Exact score — stacks with Tier 1 | **+7** (+3 bonus if result also correct = **+10** total) |
| 3 | Total corners: ≤9 / exactly 10 / ≥11 | **+4** |
| 4 | Both Teams To Score — Yes / No | **+2** |
| 5 | Over / Under 2.5 goals | **+3** |

**Maximum: 19 points per match.**

No streak bonus (removed — `streak_bonus` column still exists in DB for backward compat, always 0).

### Backward compatibility: Tier 3 was Half-Time result

Old predictions have `predicted_halftime_outcome`. New predictions have `predicted_corners`. Both fields exist on the `predictions` table. `calcBreakdown()` in `utils.ts` handles both. Never remove either field.

- Old predictions display: hardcoded string `"Half Time"` — **do not use `t('halfTimeResult')`**, that i18n key was removed
- New predictions display: corners tier label

### Source of truth

`backend/src/services/pointsEngine.ts` — pure function, no DB calls. `frontend/src/lib/utils.ts → calcBreakdown()` mirrors it client-side for prediction previews. Both must always be in sync.

### Extra time & penalties

Prediction scoring always uses **regulation-time score** (`regulation_home` / `regulation_away`).
If `regulation_home` is null, falls back to `home_score` / `away_score` (safe for non-ET matches).
`went_to_penalties = true` → shootout happened. `penalty_home` / `penalty_away` store the shootout score.

### Corners — International Friendlies exception

Corners (`predicted_corners`) is **hidden** in `PredictionForm` for league 4396 (International Friendlies).
`corners_total` must be set manually in Supabase, and friendlies have 50+ matches/day.

```typescript
// PredictionForm.tsx
const LEAGUES_WITHOUT_CORNERS = new Set([4396])
```

---

## 11. Coin Economy

### Earn & spend

| Event | Coins |
|-------|-------|
| Join bonus (one-time) | **+120** |
| Daily login bonus | **+30** |
| Stake a prediction | **−(sum of tiers played)** |
| Tier resolved correctly | **+(points_earned × 2)** |

### Coin costs per tier (`constants.ts → COIN_COSTS`)

```typescript
RESULT_ONLY: 3,   // predicted outcome only (no exact score)
SCORE: 10,        // predicted home + away score (includes result cost)
CORNERS: 4,
BTTS: 2,
OVER_UNDER: 3,
JOIN_BONUS: 120,
DAILY_BONUS: 30,
MAX_PER_MATCH: 19,
```

### UX rules — never violate

- **Never show negative coin balance.** Clamp at 0.
- **Always show `+coinsBack`** — `coinsBack = pointsEarned × 2`. Always ≥ 0.
- **Only show a "profit" line** when `coinsBack > coinsBet`.

### Daily bonus timezone

`claim_daily_bonus` DB function uses `(NOW() AT TIME ZONE 'Asia/Jerusalem')::DATE`, not `CURRENT_DATE`. The daily window resets at midnight Israel time.

### Key DB functions

```sql
-- Fully IDEMPOTENT after migration 032. Calling this 100× with the same args
-- produces the same result as calling it once. The function tries to INSERT
-- the coin_transactions log row first with ON CONFLICT DO NOTHING against the
-- partial unique index `coin_transactions_bet_won_unique`. If the insert wins,
-- group_members.coins is credited and balance_after is backfilled. If the
-- insert loses (duplicate), the function returns the current balance unchanged.
-- This is the race-condition fix — concurrent score resolvers (GitHub Actions
-- cron + Render scheduler + manual sync) cannot double-credit the same award.
increment_coins(
  p_user_id     UUID,
  p_group_id    UUID,
  p_match_id    UUID,
  p_amount      INTEGER,
  p_description TEXT        DEFAULT 'Prediction won',
  p_created_at  TIMESTAMPTZ DEFAULT NULL  -- when omitted, falls back to NOW()
) RETURNS INTEGER  -- new (or unchanged) balance

claim_daily_bonus(user_id UUID) → BOOLEAN  -- true = claimed, false = already claimed today
```

### Coin transaction created_at — "the times are sacred"

When `scoreUpdater.ts` resolves a prediction, it passes `p_created_at = matchEndAt` (kickoff_time + 105 min) to `increment_coins`. The same `matchEndAt` is also written to the `notifications` and `group_events` rows for that resolution. This means:

- A user who wins coins from a 21:00 match always sees `21:45` (or thereabouts) as the transaction/notification time, regardless of when the backend actually processed it
- Cold-start delays, GitHub Actions schedule jitter, and catch-up resolutions all produce the same "real" time
- The user's words, repeated verbatim: **"גם אם שחקן לא נמצא בתוך האפליקציה עדיין הזמנים הם קודש"** ("even if a player isn't in the app, the times are still sacred")

Never use `NOW()` for derived rows in resolution code paths. Always thread `matchEndAt` through.

---

## 12. Match Status System

### Status strings (stored in DB)

| Status | Meaning | Display |
|--------|---------|---------|
| `NS` | Not started | "Upcoming" |
| `1H` | First half live | "Live" (green) |
| `HT` | Half time | "Half Time" (yellow) |
| `2H` | Second half live | "Live" (green) |
| `ET1` | Extra time first half | "ET Live" (amber) |
| `ET2` | Extra time second half | "ET Live" (amber) |
| `AET` | After extra time (no pens) | "AET" (amber) |
| `PEN` | Penalty shootout live | "Pens" (amber) |
| `FT` | Final | "Full Time" (muted) |
| `PST` | Postponed | "Postponed" (red) |
| `CANC` | Cancelled | "Cancelled" (red) |

### Frontend-only sentinel statuses (never stored in DB)

| Sentinel | When used | Display |
|----------|-----------|---------|
| `DELAYED` | NS match past kickoff, ESPN still shows as pre/scheduled | "Delayed" (orange) |
| `SYNCING` | `isSyncing=true` AND underlying status is `DELAYED` | "Syncing…" (blue) |
| `ET_HT` | Live break between ET halves | "AET HT" (amber) |

Computed in `MatchCard.tsx`, passed to `MatchStatusBadge` as props. Never write them to the database.

**`SYNCING` intercept logic** (in `MatchStatusBadge`):
```typescript
const effectiveStatus = isSyncing && status === 'DELAYED' ? 'SYNCING' : status
```
This prevents the alarming orange "Delayed" badge from flashing during cold start before ESPN data arrives.

### Stalled NS matches (`isPastKickoffNS`)

```typescript
const isPastKickoffNS = match.status === 'NS' && new Date(match.kickoff_time).getTime() < Date.now()
```

These show: animated `— —` score, `~{minutesSinceKickoff}'` clock, **"Delayed"** badge (orange — never green "Live").

### Tab filtering rules

- **Results tab**: `status = 'FT'` only — never PST or CANC
- **Live tab**: `['1H', 'HT', '2H']` + stalled-NS within 3-hour buffer
- **All tab**: live → upcoming NS → completed FT (sorted in that priority)

### MatchTimeline

- Fetches ESPN `summary?event={id}` client-side for FT matches
- Returns `null` entirely when ESPN has no `keyEvents` (friendlies, small-nation matches)
- **Never** shows "No event data available" — the section is hidden entirely

---

## 13. League System & ESPN Coverage

### Internal ID → ESPN slug mapping

Must be kept in sync between **both** files:
- Backend: `backend/src/services/espn.ts → LEAGUE_ESPN_MAP`
- Frontend: `frontend/src/lib/constants.ts → LEAGUE_ESPN_SLUG`

| League | Internal ID | ESPN Slug |
|--------|-------------|-----------|
| Premier League | 4328 | `eng.1` |
| La Liga | 4335 | `esp.1` |
| Bundesliga | 4331 | `ger.1` |
| Serie A | 4332 | `ita.1` |
| Ligue 1 | 4334 | `fra.1` |
| Champions League | 4346 | `uefa.champions` |
| Europa League | 4399 | `uefa.europa` |
| Conference League | 4877 | `uefa.europa.conf` |
| FA Cup | 9001 | `eng.fa` |
| League Cup (Carabao) | 9002 | `eng.league_cup` |
| Copa del Rey | 9003 | `esp.copa_del_rey` |
| International Friendlies | 4396 | `fifa.friendly` |
| UEFA Nations League | 4635 | `uefa.nations` |
| World Cup Qualifiers 2026 | 5000 | `uefa.worldq` |

### Leagues with NO ESPN coverage (silently skipped)

- **World Cup** (4480) — only relevant during tournament years
- **Euro Championship** (4467) — only relevant every 4 years

### Removed leagues

- **Israeli Premier League** (4354 / `isr.1`) — removed April 2026. ESPN only covers the 2024-25 season; all 2026-date queries return 0 events. API-Football free plan blocks season 2025. Removed from `FOOTBALL_LEAGUES`, `LEAGUE_ESPN_SLUG`, and `LEAGUE_ESPN_MAP` entirely.

### Adding a new league

```bash
# 1. Verify the ESPN slug works
curl "https://site.api.espn.com/apis/site/v2/sports/soccer/{slug}/scoreboard"

# 2. Add to frontend
# frontend/src/lib/constants.ts → FOOTBALL_LEAGUES + LEAGUE_ESPN_SLUG

# 3. Add to backend
# backend/src/services/espn.ts → LEAGUE_ESPN_MAP
```

---

## 14. Database & Migrations

Migrations live in `supabase/migrations/`. Current sequence: **001 → 033** (024 and 025 do not exist).
Apply via `supabase db push --linked` (auto-runs via hook on migration file write once logged in).

| Migration | What it adds |
|-----------|-------------|
| `001` | Initial schema: profiles, groups, group_members, matches, predictions, leaderboard |
| `002` | RLS policies |
| `003` | Functions & triggers: `increment_coins`, `claim_daily_bonus`, leaderboard update trigger, `set_updated_at` |
| `004` | Bug fixes |
| `005` | `display_clock` column on matches |
| `006` | Leaderboard readable by all group members |
| `007` | Prediction resolution fixes |
| `008` | `last_week_points` column on leaderboard |
| `009` | Streak bonus backfill |
| `010` | Full data repair |
| `011` | `reset_group_scores` RPC |
| `012` | `streak_bonus` column (always 0, kept for backward compat) |
| `013` | `halftime_pts` column |
| `014` | `predicted_corners` column (replaces `predicted_halftime_outcome` for new predictions) |
| `015` | `regulation_home` / `regulation_away` columns |
| `016` | `went_to_penalties` boolean |
| `017` | Predictions delete policy (idempotent — uses DROP POLICY IF EXISTS) |
| `018` | `penalty_home` / `penalty_away` columns |
| `019` | `red_cards_home` / `red_cards_away` columns |
| `020` | Coins system: `user_coins` table, `increment_coins`, `claim_daily_bonus` |
| `021` | Coins bug fixes |
| `022` | Admin features: group delete policy, group_members delete policy (idempotent) |
| `023` | Admin security RPCs: `is_super_admin`, `admin_get_stats`, `admin_get_users`, `admin_get_groups`, `admin_get_user_coins`, `admin_adjust_coins`, `admin_update_username`, `admin_delete_group`, `admin_delete_user_data`, `admin_rename_group` |
| `026` | Persistent notifications table + RLS |
| `027` | Fix `admin_delete_user_data` to also wipe `coin_transactions` |
| `028` | Group activity feed (`group_events` table for The Locker Room) |
| `029` | `group_events` FK → `profiles(id)` so PostgREST joins work |
| `030` | `increment_coins` accepts explicit `p_created_at` so coin txns reflect match end time, not server clock ("the times are sacred") |
| `031` | One-off cleanup of duplicate `coin_transactions` rows + rebuild `group_members.coins` from cleaned ledger (race-condition aftermath) |
| `032` | **Bulletproof coin dedup**: partial unique index on `coin_transactions (user_id, group_id, match_id, description) WHERE type='bet_won'` + rewrite `increment_coins` as fully idempotent (`INSERT … ON CONFLICT DO NOTHING` first, then credit balance only if insert won the race) |
| `033` | Cleanup of duplicate `group_events` WON_COINS rows + duplicate `notifications` prediction_result rows + matching partial unique indexes (`group_events_won_coins_unique`, `notifications_prediction_result_unique`) as DB-level backstops |

### Migration idempotency

All migrations from 017 onward use `DROP … IF EXISTS` before `CREATE` so they can be re-applied safely without errors. If the Supabase CLI records a migration as applied despite a partial failure, use:

```bash
supabase migration repair --linked --status reverted <version>
supabase db push --linked
```

### CI migration repair

`ci.yml` auto-repairs Supabase migration history for migrations 001–014. This handles the case where migrations were applied manually before the CI was set up.

### Key tables (abbreviated)

**`profiles`** — `id` (FK → auth.users), `username` (unique), `avatar_url`, `group_id`, `created_at`

**`matches`** — `id`, `external_id` (ESPN event ID), `league_id`, `home_team`, `away_team`, `home_team_badge`, `away_team_badge`, `kickoff_time`, `status`, `home_score`, `away_score`, `regulation_home`, `regulation_away`, `went_to_penalties`, `penalty_home`, `penalty_away`, `halftime_home`, `halftime_away`, `corners_total`, `red_cards_home`, `red_cards_away`, `display_clock`, `season`, `round`, `updated_at`

**`predictions`** — `id`, `user_id`, `match_id`, `group_id`, `predicted_outcome` (H/D/A), `predicted_home_score`, `predicted_away_score`, `predicted_halftime_outcome` (legacy), `predicted_corners` (new), `predicted_btts`, `predicted_over_under` (over/under), `points_earned`, `is_resolved`, `created_at`

**`leaderboard`** — `id`, `user_id`, `group_id`, `total_points`, `weekly_points`, `last_week_points`, `predictions_made`, `correct_predictions`, `streak_bonus` (always 0), `updated_at`

**`group_members`** — `user_id`, `group_id`, `coins` ← **this is the authoritative coin balance**, `joined_at`

**`user_coins`** — `id`, `user_id`, `group_id`, `coins`, `last_daily_claim`, `updated_at` (supplementary — created in 020)

### RLS summary

- **profiles**: readable by all; writable by owner (`auth.uid() = id`)
- **matches**: public read; service role writes only (backend)
- **predictions**: readable by group members; writable by owner; NOT readable by others when `status = 'NS'` (pre-kickoff privacy)
- **leaderboard**: readable by all group members; service role writes
- **group_members**: readable by group members; service role writes coins

---

## 15. Theme System

**Version: GoalBet v2.0 — "Cold Sea Navy / Frost"**

Dark mode is the default. Light mode toggled via `ThemeToggle` in the top bar.

**Mechanism:** `html.light` class on `<html>` managed by `themeStore.ts`.
**CSS tokens:** defined in `frontend/src/index.css` under `:root` (dark) and `html.light` (light overrides).

### Design tokens — Dark ("Cold Sea Navy")

```css
--color-bg-base: #0a1733              /* Deep Navy */
--color-bg-card: rgba(15,40,84,0.4)   /* Rich navy glass */
--color-accent-green: #BDE8F5         /* Bright Ice Blue — primary accent */
--color-accent-secondary: #4988C4    /* Steel Blue */
--color-accent-orange: #FF3366
--color-text-primary: #FFFFFF
--color-text-muted: rgba(189,232,245,0.60)
--color-border-subtle: rgba(73,136,196,0.20)
--color-border-bright: rgba(189,232,245,0.40)
--glow-green: 0 0 24px rgba(73,136,196,0.40)
```

Body background: navy `#0F2854` ellipse from top + steel blue blooms at bottom corners, fixed attachment.
`body::after`: tactical dot matrix (`24×24px` radial dots at `rgba(189,232,245,0.10)`), masked to fade at top/bottom edges, `z-index: -1`.

### Design tokens — Light ("Frost")

```css
--color-bg-base: #F2F6FA             /* Icy sky */
--color-bg-card: #FFFFFF
--color-accent-green: #1C4D8D        /* Deep Navy */
--color-accent-secondary: #4988C4
--color-text-primary: #0A1733
--color-text-muted: #4988C4
--color-border-subtle: rgba(15,40,84,0.08)
```

Body background: white ellipse bloom from top + ice-blue corner.
`body::after`: navy dot matrix (`rgba(15,40,84,0.05)`), same grid/mask as dark.

### Fonts

- **`font-display`** / **`font-sans`** / **`font-dm`**: **Inter** + Heebo fallback — all UI body text and numbers
- **`font-barlow`** / **`font-headline`**: Barlow Condensed — labels, stat headers
- **`font-bebas`**: Bebas Neue — score display
- **`font-mono`**: SF Mono / Roboto Mono

### Bento card classes (ProfileBentoV2)

Use semantic CSS classes — never hardcode rgba in bento components:

| Class | Dark | Light |
|-------|------|-------|
| `bento-card-accent` | volt tint | navy tint |
| `bento-card-purple` | purple tint | steel blue tint |
| `bento-card-default` | white/3 | white/90 |
| `bento-hero-card` | volt border | navy border |

### Hover effects

- **Match cards** (`MatchCard`): diagonal shimmer sweep on hover entry. No tilt — preserves prediction form UX.
- **Profile bento** (`TiltCardV2`): 3° tilt with spring physics. No glare overlay.
- **Buttons** (`MagneticButtonV2`): magnetic pull within 80px radius. Variants: `volt`, `ghost`, `purple`.

### Light mode contrast overrides

`html.light` in `index.css` remaps `text-white/*` and `border-white/*` opacity variants to navy equivalents. The opacity floors were raised in Sprint 14/15 to improve contrast — do not revert them:

| Tailwind class | Light mode minimum opacity |
|---|---|
| `border-white/8` → `border-white/20` | 0.15 |
| `border-white/15` → `border-white/25` | 0.20 |
| `text-white/25` | 0.45 (was 0.30) |
| `text-white/30` | 0.50 (was 0.38) |

If a new `text-white/XX` or `border-white/XX` class appears invisible in light mode, add an override to the `html.light` block in `index.css`.

### League logos — ESPN dark/light variants

ESPN CDN provides two logo sets:
- Light logos: `https://a.espncdn.com/i/leaguelogos/soccer/500/{id}.png`
- Dark logos: `https://a.espncdn.com/i/leaguelogos/soccer/500-dark/{id}.png`

`MatchCard.tsx` renders **both** `<img>` tags and toggles visibility via CSS:

```css
/* index.css — dark mode (default): show dark variant */
.league-logo-light { display: none; }
.league-logo-dark  { display: inline-block; }

/* Light mode: swap */
html.light .league-logo-light { display: inline-block; }
html.light .league-logo-dark  { display: none; }
```

ESPN logo IDs are in `constants.ts → FOOTBALL_LEAGUES[].espnLogoId`. Set to `null` for leagues with no ESPN logo (World Cup Qualifiers, Euro Championship). Verified working IDs: Premier League=23, La Liga=15, Bundesliga=10, Serie A=12, Ligue 1=9, Champions League=2, Europa League=2310, Conference League=20296, FA Cup=40, League Cup=41, Copa del Rey=80, Nations League=2395, Friendlies=53.

**Do not use Tailwind `dark:` prefix** — this project uses `html.light` class, not Tailwind's dark mode. Use the CSS class toggle pattern above.

### Live match animations (index.css)

| CSS class | Effect | Where used |
|-----------|--------|------------|
| `.animate-live-breathing` | Green glow pulse on card border (opacity 0.20→0.40) + border-color pulse (0.25→0.45) | `MatchCard` — applied to `GlassCard` when match is live |
| `.goal-flash` | Green overlay flash on goal scored | `MatchCard` — applied via `cardRef` when score changes |
| `.pitch-grass` | Subtle dark green gradient background | `TacticalPitch` — pitch surface |

Score flip animation uses Framer Motion `AnimatePresence mode="popLayout"` with spring scale (1.3→1→0.8).

### Rules

- **Never hardcode dark hex colors** in modals, tooltips, or cards — they break in light mode
- **Never use Tailwind `dark:` prefix** — this project uses `html.light` class toggle, not Tailwind dark mode
- Use `card-elevated` CSS class instead of raw hex backgrounds
- Use CSS vars (`var(--color-tooltip-bg)`) for dynamic surfaces
- For new bento/stat cards: add a `bento-*` CSS class pair, not inline rgba
- All `text-white/*` opacity variants have `html.light` overrides mapping to navy — if adding a new opacity variant, add it to both sets
- `text-white/85` and below all have overrides; if a new variant is invisible in light mode, add `html.light .text-white\/XX` to `index.css`

---

## 16. Internationalisation (i18n)

All UI strings live in `frontend/src/lib/i18n.ts` as `en` and `he` objects.

```typescript
const { t } = useLangStore()
t('matchDay')  // → "Match Day" or "יום משחק"
```

`TranslationKey` type is auto-derived from the `en` object — TypeScript errors on unknown keys.

- Language stored in `langStore.ts` (persisted to localStorage)
- RTL layout applied automatically via `useRTLDirection` hook (`document.dir`)
- Use `ms-` / `me-` instead of `ml-` / `mr-` for RTL compatibility

### Rules

- `t()` accepts **only** `TranslationKey` — passing a plain `string` is a TS error
- Import `TranslationKey` from `../../lib/i18n` when calling `t` in helper functions
- **`halfTimeResult` key re-added** (en: `"Half Time"`, he: `"מחצית"`). Use `t('halfTimeResult')` for the backward-compat HT tier label
- **Every visible UI string must go through `t()`** — including `aria-label`, `title`, `placeholder`, and share text. Hardcoded English strings silently break Hebrew mode.

### Parameterized translations

Some keys contain a `{0}` placeholder. These are not template literals — use `.replace()`:

```typescript
// ✅ Correct
t('secsAgo').replace('{0}', String(elapsed))   // → "לפני 12 שניות"
t('minsAgo').replace('{0}', String(minutes))   // → "לפני 3 דקות"

// ❌ Wrong — t() does not interpolate automatically
t(`secsAgo`, { count: elapsed })
```

Always add the key to **both** `en` and `he` blocks in `i18n.ts`. `TranslationKey` is derived from `en` — a missing `en` key causes a TypeScript error; a missing `he` key silently falls back to the `en` value.

### Hebrew football terminology

| English | Correct Hebrew |
|---------|---------------|
| Corners | **קרנות** (not ~~קורנרים~~) |
| Score | **סקור** (not ~~ניקוד~~) |
| FT Result hit rate | **ניחוש תוצאה** |

---

## 17. Stores

All stores use Zustand. Persistence uses `localStorage` where noted.

| Store | Key state | Persistence | Notes |
|-------|-----------|-------------|-------|
| `authStore.ts` | `user`, `profile`, `session`, `loading`, `initialized` | Session-based | `init()` must be called once on app mount |
| `groupStore.ts` | `groups[]`, `activeGroupId` | localStorage | `fetchGroups(userId)` on every login |
| `coinsStore.ts` | `coins`, `lastDailyBonus` | Synced from DB | `initCoins(userId, groupId)` re-checks on tab focus for midnight reset |
| `langStore.ts` | `lang` (`'en'` \| `'he'`) | localStorage | Also controls `document.dir` via `useRTLDirection` |
| `themeStore.ts` | `theme` (`'dark'` \| `'light'`) | localStorage | Manages `html.light` class |
| `uiStore.ts` | `activeModal`, `toasts[]`, `isSyncing` | Memory only | `openModal(id)`, `addToast(msg, type)`, `setSyncing(bool)` |

---

## 18. Hooks

| Hook | Purpose |
|------|---------|
| `useMatches.ts` | Fetches match list from Supabase. Listens for `goalbet:synced` + Supabase Realtime. Handles tab/all/live/completed filters |
| `useMatchSync.ts` | **Manual sync ONLY** (Settings "Sync Now" button). 60s timeout. Never add auto-trigger |
| `usePredictions.ts` | Fetches and saves user predictions for a set of match IDs. Optimistic updates |
| `useGroupMatchPredictions.ts` | Fetches all group members' predictions for a specific match (social display, H2H) |
| `useLeaderboard.ts` | Fetches leaderboard for active group + Realtime subscription |
| `useLiveClock.ts` | Ticking clock for live matches. Increments from `display_clock` value |
| `useNewPointsAlert.ts` | Detects newly earned points since last visit, fires a toast |
| `useAuthV2.ts` | Auth state machine for `AuthContainer`. 8 views: `email → signin \| signup → oauth-merge \| forgot → check-email \| set-password → success`. Detects `?type=recovery` for password reset |
| `useAuth.ts` | Legacy Google OAuth wrapper — kept for backward compat, not used in main flows |
| `useRTLDirection.ts` | Sets `document.documentElement.dir` based on active language |

---

## 19. CI / GitHub Actions

### `sync-cron.yml` — every 5 minutes, 24/7

```
1. Check BACKEND_URL secret is set
2. GET  /api/health         → wake Render, retry 3× with 10s delays (30s per attempt)
3. POST /api/sync/scores    → resolve predictions + award coins (75s curl timeout) ← ALWAYS runs
4. POST /api/sync/matches   → pull ESPN fixtures (75s curl timeout, continue-on-error: true)
```

**Critical design invariant:** Score resolution (step 3) runs **before** fixture sync (step 4), and step 4 has `continue-on-error: true`. This guarantees coin payouts are never blocked by an ESPN fixture timeout. An ESPN outage delays fixture updates but never delays coin awards.

**No auth required** — sync endpoints are intentionally public. Do not add `SYNC_SECRET` back.

### `ci.yml` — every push / PR to main

1. TypeScript type-check (`tsc --noEmit`)
2. Vite build (`npm run build`)
3. Supabase migration history repair (001–014)

### GitHub Secrets

| Secret | Used by | Purpose |
|--------|---------|---------|
| `BACKEND_URL` | `sync-cron.yml` | Production Render URL |
| `SUPABASE_ACCESS_TOKEN` | `ci.yml` | Supabase CLI auth for migration repair |
| `SUPABASE_PROJECT_REF` | `ci.yml` | Supabase project ID |

`SYNC_SECRET` must **not exist** — it was removed because it caused 401s in GitHub Actions.

---

## 20. Admin Console

The admin console is accessible at `/admin` **only** for `roychen651@gmail.com`. It is invisible to all other users — the route silently redirects to `/`.

### Security model: double protection

| Layer | Mechanism |
|-------|-----------|
| Client-side | `AdminProtectedRoute` checks `user.email === 'roychen651@gmail.com'` before rendering |
| Server-side (RPCs) | Every admin SQL function starts with `IF NOT is_super_admin() THEN RAISE EXCEPTION` |
| Server-side (backend) | `resolveAdminEmail()` verifies JWT via service role; returns 403 if not admin |

**Never remove either layer.** Client-side is UX; server-side is security.

### `is_super_admin()` function

```sql
CREATE OR REPLACE FUNCTION is_super_admin()
RETURNS BOOLEAN LANGUAGE sql SECURITY DEFINER STABLE SET search_path = public AS $$
  SELECT (auth.jwt() ->> 'email') = 'roychen651@gmail.com';
$$;
```

All admin RPCs call this as their first action. They are `SECURITY DEFINER` — they run with the Supabase service role's privileges, so the email check is the only access gate.

### Admin RPCs (migration 023)

| Function | Purpose |
|----------|---------|
| `admin_get_stats()` | Platform KPIs: total users, groups, matches, predictions, coins circulating |
| `admin_get_users()` | All users with email, username, group count, total coins |
| `admin_get_groups()` | All groups with admin email, member count, invite code |
| `admin_get_user_coins(user_id)` | Per-group coin balance for a specific user |
| `admin_adjust_coins(user_id, group_id, delta)` | Add or subtract coins (floor at 0), logs to coin_transactions |
| `admin_update_username(user_id, username)` | Rename any user |
| `admin_rename_group(group_id, name)` | Rename any group |
| `admin_delete_group(group_id)` | Cascade-delete group and all members/predictions |
| `admin_delete_user_data(user_id)` | Wipe all public-schema rows for user (call before backend delete) |

### Admin backend routes

| Method | Path | Purpose |
|--------|------|---------|
| `DELETE` | `/api/admin/users/:userId` | Hard-delete user from `auth.users` (service role) |
| `POST` | `/api/admin/reset-password` | Send password reset email via Supabase admin API |

Both require a valid `Authorization: Bearer <jwt>` from `roychen651@gmail.com`. The `resolveAdminEmail()` middleware calls `supabaseAdmin.auth.getUser(token)` to verify identity server-side.

### User deletion flow (two-step)

```
1. Frontend: supabase.rpc('admin_delete_user_data', { p_user_id })
   → wipes all public-schema rows (predictions, group_members, leaderboard, etc.)

2. Frontend: fetch('DELETE /api/admin/users/:userId', { Authorization: Bearer jwt })
   → backend calls supabaseAdmin.auth.admin.deleteUser(userId)
   → removes the auth.users row permanently
```

Step 1 **must complete before** step 2. Reversing the order leaves orphaned data.

### Admin UI components

**`AdminDashboardPage`** (`pages/admin/AdminDashboardPage.tsx`)
- Calls `admin_get_stats()` on mount
- Bento grid of 5 animated KPI cards (staggered Framer Motion fade-in)
- System health section: Force Score Sync button, Sync Fixtures button, Health Check link

**`UserManagement`** (`components/admin/UserManagement.tsx`)
- Search filter on email / username
- Staggered row animations (`delay: i * 0.025`)
- Edit name → `admin_update_username` RPC
- Manage coins → `admin_get_user_coins` → per-group `admin_adjust_coins`
- Reset password → `POST /api/admin/reset-password`
- Delete → `admin_delete_user_data` RPC then `DELETE /api/admin/users/:id`

**`GroupManagement`** (`components/admin/GroupManagement.tsx`)
- Search filter on name / admin email / invite code
- Rename → `admin_rename_group` RPC
- View members → `group_members` direct query with coins per member
- Delete → `admin_delete_group` RPC wrapped in `DangerModal`

**`DangerModal`** (`components/admin/DangerModal.tsx`)
- Spring animation (`y: 40 → 0`)
- Requires typing `"DELETE"` exactly to enable the confirm button
- Used for both group deletion and user deletion

**`AdminLayout`** (`components/admin/AdminLayout.tsx`)
- Desktop: fixed sidebar with logo + "ADMIN" badge, nav links, "Back to App" button
- Mobile: fixed top bar with nav links
- Renders `<Outlet />` for nested admin routes

---

## 21. Common Pitfalls

### Sync

- **Never add auto-sync to page components.** `AppShell` owns all automatic sync. Two auto-triggers create race conditions.
- **Never remove `AbortController` timeouts** from sync fetches. Without them, the UI hangs indefinitely on Render cold starts (45–60s).
- **Score sync timeout must be ≥ 75s.** Render cold start takes 45–60s. A shorter timeout means score sync always fails on cold start, delaying coin payouts until the next cron run.
- **Score resolution must run before fixture sync in `sync-cron.yml`.** A failed ESPN call must never block coin payouts. `continue-on-error: true` on the fixture step is intentional — never remove it.
- **Adding a league to `FOOTBALL_LEAGUES` without adding it to `LEAGUE_ESPN_MAP`** silently produces no data — no error, no log. Always add to both.
- **Wrong ESPN slug returns 400.** Verify: `curl "https://site.api.espn.com/apis/site/v2/sports/soccer/{slug}/scoreboard"`

### Match display

- **PST / CANC must never appear in Results or Live tabs.** Only `FT` is shown in completed tabs.
- **Stalled NS matches show "Delayed" (orange), not "Live" (green).** Sentinel is `'DELAYED'`, never `'1H'`.
- **MatchTimeline returns null when ESPN has no events.** Do not add an empty state — the section is hidden.
- **League logos use dual `<img>` tags** (dark + light variant), toggled via `.league-logo-dark` / `.league-logo-light` CSS classes. Never use a single `<img>` with a CSS `filter` hack — it doesn't match ESPN's official dark variants.
- **ESPN logo IDs must be verified.** Wrong IDs return 404 silently (broken image). Check `constants.ts → FOOTBALL_LEAGUES[].espnLogoId`. Set `espnLogoId: null` for leagues without ESPN logos.

### Predictions & scoring

- **Old predictions have `predicted_halftime_outcome`; new ones have `predicted_corners`.** Both coexist. `calcBreakdown()` handles both.
- **Scoring always uses `regulation_home` / `regulation_away`** for ET/penalty matches.
- **Corners are hidden for league 4396** (International Friendlies). Do not remove `LEAGUES_WITHOUT_CORNERS`.
- **`halfTimeResult` key re-added** — use `t('halfTimeResult')` for the old HT tier label.

### Auth

- **`LoginPage` is a thin wrapper only.** Auth UI and logic belong in `AuthContainer` + `useAuthV2`. Do not add auth logic to `LoginPage`.
- **Identity collision signal** is `signUp()` returning "User already registered" (HTTP 422). Do not add a "check email" API endpoint — that breaks anti-enumeration.
- **Forgot password always shows success.** `handleForgotPassword` swallows all errors. Never show "email not found".
- **Password recovery redirects to `/login?type=recovery`**, not `/auth/callback`. `useAuthV2` detects this on mount.
- **`PasswordStrength` is shared** between `AuthContainer` and `SettingsPage`. Import from `auth-v2/PasswordStrength`; never duplicate it.

### UI / Components

- **`Avatar` expects `emoji:🏆` format** (with `emoji:` prefix). Raw emoji string is treated as an image URL and silently fails.
- **Hardcoded hex colors in modals break light mode.** Use `card-elevated` class or `var(--color-tooltip-bg)`.
- **`t()` only accepts `TranslationKey`.** Passing a plain `string` is a TypeScript error.
- **Use `ms-` / `me-` for margins, never `ml-` / `mr-`.** RTL layout requires logical CSS properties.
- **Bottom-sheet modals must include `onPointerDown={e => e.stopPropagation()}` on scroll containers.** Without this, Framer Motion drag fires instead of scroll on touch devices.
- **All `<img>` tags must have numeric `width` and `height` attributes.** Omitting them causes CLS on slow connections — even when Tailwind `w-` / `h-` classes are present.
- **`ErrorBoundary` wraps `<Outlet />` in `AppShell`.** Do not remove it. It catches render errors in any page and shows a premium bilingual fallback instead of a white screen.
- **`MatchCard.tsx` is the single active card implementation.** It exports two symbols: `MatchCardCore` (private internal component) and `MatchCard` (public shimmer wrapper). `MatchCardV2.tsx` was deleted in Sprint 17; `USE_V2_CARDS` no longer exists. All card work goes into `MatchCard.tsx`.
- **The share invite string in `SettingsPage.tsx` must be localized.** It is the primary viral surface — a Hebrew user must send a Hebrew message.
- **TacticalPitch player nodes must all be full opacity.** Never use `opacity-30` or similar on subbed-out players — it makes them invisible on the glass pitch. Use the `▼` marker instead.
- **Never use Tailwind `dark:` prefix in this project.** Theme toggle uses `html.light` class, not Tailwind's dark-mode. Use CSS class pairs (e.g., `.league-logo-dark` / `.league-logo-light`) toggled via `html.light` selectors in `index.css`.

### Coins

- **Never show negative coin amounts.** Clamp at 0.
- **Daily bonus uses Israel timezone** (`Asia/Jerusalem`). `CURRENT_DATE` is wrong.
- **Friend prediction privacy:** predictions are hidden (🔒) when `status = 'NS'` AND `kickoff > now`. Never show pre-kickoff predictions to other group members.
- **H2H modal** opens when clicking ANOTHER user's leaderboard row. Clicking your own row opens `UserMatchHistoryModal`. This is intentional.
- **`increment_coins` is idempotent — do not work around it.** If a re-call appears to have done nothing, it's because the unique index already accepted that exact `(user, group, match, description)` combination. Find out which earlier call inserted it; never bypass the constraint by varying the description (e.g. appending a timestamp). That defeats the dedup and re-introduces the race the user was burned by.
- **Always pass `p_created_at = matchEndAt`** when calling `increment_coins` from a score-resolution path. The user has explicitly told us "the times are sacred" — the transaction must reflect when the match ended, not when the backend processed it. Same rule for `notifications.created_at` and `group_events.created_at` rows inserted in the same resolution block.
- **`group_members.coins` must always equal `SUM(coin_transactions.amount)`** for that user/group. If they ever diverge, the cached balance is wrong and a one-off rebuild migration (see 031) is needed. The reconciliation can be verified with: `SELECT user_id, group_id, coins, (SELECT SUM(amount) FROM coin_transactions ct WHERE ct.user_id = gm.user_id AND ct.group_id = gm.group_id) AS ledger_sum FROM group_members gm`.

### Admin

- **Never skip `is_super_admin()` in admin RPCs.** All `SECURITY DEFINER` functions run with elevated privileges — the email check is the only gate.
- **Always call `admin_delete_user_data()` before `DELETE /api/admin/users/:id`.** Reversing the order leaves orphaned rows in public schema with no foreign-key owner.
- **Admin routes on the backend use service-role JWT verification**, not anon key. Never swap `supabaseAdmin` for the regular `supabase` client in `backend/src/routes/admin.ts`.
- **`DangerModal` requires typing `"DELETE"` exactly.** Do not pre-fill or bypass the input check for "convenience".
- **Migrations that already exist in the remote history** can be force-reapplied with `supabase migration repair --linked --status reverted <version>` then `supabase db push --linked`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Roychen651) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
