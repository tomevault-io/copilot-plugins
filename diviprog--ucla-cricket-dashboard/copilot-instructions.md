## ucla-cricket-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UCLA Cricket Stats Platform - A comprehensive cricket statistics platform that parses CricClubs HTML scorecards, manages player rosters with aliases, calculates batting/bowling/fielding statistics, and displays them in a dashboard.

**Tech Stack:** Next.js 14 (App Router), TypeScript, Supabase (PostgreSQL), Tailwind CSS, Cheerio (HTML parsing), ExcelJS (Excel export)

## Development Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Run production build
npm start

# Seed database with players and seasons
npx tsx src/scripts/seed-players.ts
```

## Environment Setup

Create `.env.local` with:
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

Database schema is referenced in `src/lib/supabase/schema.sql` (run in Supabase SQL Editor).

## Architecture

### Data Flow

**Match Import:** HTML Upload → `/api/matches/import` → Parse CricClubs HTML → Generate content hash → Check for duplicates → Resolve/create players via aliases → Write match + performances to Supabase → Update season stats

**Dashboard:** Load season filters → Fetch team stats, top performers, recent matches → Display leaderboards

**Editable Scorecard:** Load match + performances → Render tables with reassign dropdowns + inline stat editing → Call reassign/update APIs → Refresh match data

### Key Components

**Authentication (`src/lib/auth/auth-context.tsx`):**
- Uses Supabase auth session management
- `isAdmin` = any authenticated user (no role-based checks)
- Provides `useAuth()` and `useRequireAdmin()` hooks
- `AdminGuard` component redirects to `/admin/login` if not authenticated

**Player Resolution System:**
- Players have canonical names in `players` table
- `player_aliases` maps name variations to canonical player IDs
- Parser resolves player names via alias lookup during import
- Import route calls `resolvePlayerName()` → checks aliases → returns playerId
- New players created via `createPlayerIfNotExists()`
- Player cache initialized via `initializePlayerCache()` for performance

**Season Detection:**
- `detectSeasonFromDate()` determines season (e.g., "2024-2025") from match date
- Seasons created automatically if missing via `getOrCreateSeason()`
- Season stats tracked separately in `player_season_stats`, `bowling_season_stats`, `fielding_season_stats`

**Stats Calculation:**
- After importing performances, `updatePlayerSeasonStats()`, `updateBowlingSeasonStats()`, `updateFieldingSeasonStats()` recalculate aggregates
- Stats computed from all performances for a player in a season
- Calculates: batting average, strike rate, boundary %, bowling economy, dot ball %, etc.

**Duplicate Detection:**
- `generateContentHash()` creates hash from HTML content
- Hash stored in `import_history` table
- Import route checks hash before processing to prevent duplicate imports

### Innings and Team Detection

**Our Team vs Opponent:**
- Parser identifies "our" team (default: "UCLA") via `metadata.ourTeamName` or name matching
- `ourInnings` = innings where batting team name contains our team name
- `opponentInnings` = other innings
- Our batting performances come from `ourInnings.battingEntries`
- Our bowling performances come from `opponentInnings.bowlingEntries` (when opponent bats, we bowl)
- Our fielding performances extracted from opponent's dismissals

**Result Calculation:**
- Result determined by comparing `ourInnings.total` vs `opponentInnings.total`
- Ignores unreliable result text from parsed HTML
- Values: 'win', 'loss', 'tie', 'no_result'

**Extras Tracking:**
- Match stores `our_extras_total`, `our_extras_wides`, `our_extras_no_balls`, `our_extras_byes`, `our_extras_leg_byes`
- Parsed from `ourInnings.extrasBreakdown`

### Fielding Stats Extraction

Fielding performances (catches, run outs, stumpings) are automatically parsed from dismissal text in batting entries:
- "c Naman S b Tanmay D" → catch credited to Naman S
- "run out (Devansh M)" → run out credited to Devansh M
- "st John K b Rohan M" → stumping credited to John K
- Parser groups by player and creates/updates `fielding_performances`

### Bowler Wicket Types

Tracks HOW bowlers take wickets via `bowler_wicket_types` table:
- Dismissal types: 'caught', 'bowled', 'lbw', 'stumped', 'hit_wicket', 'other'
- Parsed from opponent batting dismissals
- Grouped by bowler name and dismissal type
- Upserted with conflict resolution on `match_id,bowler_player_id,dismissal_type`

### Type System (`src/types/models.ts`)

**Database Models:** Season, Player, PlayerAlias, Match, ImportHistory, BattingPerformance, BowlingPerformance, FieldingPerformance, PlayerSeasonStats, BowlingSeasonStats, FieldingSeasonStats, MatchPlayerOverride

**Parsed Types:** ParsedBattingEntry, ParsedBowlingEntry, ParsedFieldingEntry, ParsedBowlerWicketEntry, ExtrasBreakdown, ParsedMatchData

**API Response Types:** PlayerWithStats, MatchWithPerformances, CalculatedBattingStats, CalculatedBowlingStats, CalculatedFieldingStats

### Utility Functions (`src/lib/utils.ts`)

- `formatScore(runs, balls, notOut)` - Format as "22(21)" or "10(6)*"
- `formatStat(value, decimals)` - Format numbers with proper decimals, handles NaN/Infinity
- `safePercentage(numerator, denominator)` - Calculate percentage with zero-division safety
- `formatDate(date)` - Format as "Dec 4, 2024"
- `getInitials(name)` - Extract first two initials
- `isUnclaimed(name)` - Check if player name is unclaimed placeholder
- `displayPlayerName(name)` - Show "Unclaimed" instead of "Unclaimed #1"
- `cn(...inputs)` - Merge Tailwind classes via clsx + tailwind-merge

## Implementation Status

### Implemented

- **`POST /api/matches/import`** - Full import pipeline (depends on missing libs listed below)
- **Dashboard page** (`src/app/page.tsx`) - Layout and UI complete (depends on missing APIs)
- **Navigation** - Header with auth state, admin links
- **Auth context** - Session management, sign in/out, admin guard
- **Editable scorecard component** - Player reassign + inline stat editing (depends on missing APIs)
- **Types & utilities** - Complete type definitions and helper functions

### Missing Dependencies (blocking build/runtime)

**Required Libraries:**
- `@/lib/supabase/client` - Supabase browser client
- `@/lib/supabase/schema.sql` - Database schema DDL
- `@/lib/parsers/cricclubs-parser` - `parseCricClubsScorecard()`, `generateContentHash()`
- `@/lib/parsers/player-resolver` - `resolvePlayerName()`, `createPlayerIfNotExists()`, `initializePlayerCache()`
- `@/lib/services/stats-service` - `detectSeasonFromDate()`, `getOrCreateSeason()`, `updatePlayerSeasonStats()`, `updateBowlingSeasonStats()`, `updateFieldingSeasonStats()`
- `@/components/ui/stats-card` - StatsCard component for dashboard

**Missing API Routes:**
- `GET /api/seasons` - Returns `{ success, seasons, competitions }` for season/competition filters
- `GET /api/stats/dashboard?seasonId=&competition=` - Returns `{ success, stats, topPerformers, recentMatches }`
- `GET /api/export?format=xlsx` - Excel export with 4 sheets (batting, bowling, fielding, matches)
- `GET /api/players/list` - Returns player list for reassign dropdown
- `POST /api/performances/reassign` - Body: `{ performanceId, performanceType, newPlayerId, matchId }`
- `POST /api/performances/update` - Body: `{ performanceId, performanceType, matchId, updates }`

**Missing Pages:**
- `/upload` - Upload CricClubs HTML scorecard
- `/players` - Batting stats/leaderboard
- `/bowling` - Bowling stats/leaderboard
- `/fielding` - Fielding stats/leaderboard
- `/matches` - Match list
- `/matches/[id]` - Match detail + editable scorecard
- `/admin/login` - Admin sign-in page
- `/players/manage` - Player/alias management

## Database Schema Notes

**Key Tables:**
- `seasons` - Academic year seasons (2024-2025, etc.)
- `players` - Canonical player records
- `player_aliases` - Name variations → player_id mapping
- `matches` - Match metadata, scores, result, extras breakdown
- `import_history` - Deduplication via content_hash
- `batting_performances` - Per-match batting stats
- `bowling_performances` - Per-match bowling stats
- `fielding_performances` - Per-match fielding stats
- `bowler_wicket_types` - How bowlers take wickets (caught/bowled/lbw/etc.)
- `player_season_stats` - Aggregated batting stats per season
- `bowling_season_stats` - Aggregated bowling stats per season
- `fielding_season_stats` - Aggregated fielding stats per season
- `match_player_overrides` - Manual player reassignments (for fixing misidentified players)

## Important Notes

**Authentication:**
- Current implementation: any authenticated Supabase user = admin
- No role-based access control or RLS policies documented
- AdminGuard redirects to `/admin/login` for unauthenticated users

**Seed Script:**
- Located at `src/scripts/seed-players.ts`
- Creates players, aliases, and seasons (2024-2025, 2025-2026)
- Falls back to hardcoded Supabase credentials if env vars missing (insecure for production)

**Name Variations:**
- Player names in scorecards vary ("Devansh M", "Devansh Mishra", "D Mishra")
- Alias system maps all variations to canonical player
- Parser must call `resolvePlayerName()` before creating performances
- Unclaimed players use special naming: "Unclaimed #1", "Unclaimed #2", etc.

**CricClubs HTML Format:**
- Parser expects specific HTML structure from CricClubs scorecards
- Extracts: teams, date, venue, innings (batting/bowling/fielding entries), extras breakdown
- Handles variations in HTML structure (bowling entries may be in either innings)

**Deployment:**
- Designed for Vercel deployment
- Environment variables must be set in Vercel dashboard
- Database runs on Supabase (free tier supported)

---
> Source: [diviprog/ucla_cricket_dashboard](https://github.com/diviprog/ucla_cricket_dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
