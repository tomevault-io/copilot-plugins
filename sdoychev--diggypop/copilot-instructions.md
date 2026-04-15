## diggypop

> Analytics dashboard for DiggyPop — a children's game platform with three game types: **Charades**, **MojiMatch**, and **Sound Challenge**. Displays usage data and generates personalized play recommendations using a rule-based engine.

# DiggyPop Advisor

Analytics dashboard for DiggyPop — a children's game platform with three game types: **Charades**, **MojiMatch**, and **Sound Challenge**. Displays usage data and generates personalized play recommendations using a rule-based engine.

## Tech Stack

- **React 18** (no TypeScript) with Vite 6
- **No component library** — all UI is inline styles
- **No state management** — just `useState` + `useMemo`
- **No router** — tab-based navigation via state
- **No backend calls** — data is hardcoded arrays (exported from Supabase)
- Dev server: `npm run dev` (port 3000, auto-opens browser)

## Project Structure

```
src/
  main.jsx          # React 18 createRoot entry
  App.jsx           # THE ENTIRE APP — all logic, data, and UI (~512 lines)
  index.css         # Minimal CSS reset only
public/
  DiggyPop_Recommendation_Algorithm.pdf
```

This is a **single-file app**. Everything lives in `src/App.jsx`.

## App.jsx Anatomy (top to bottom)

### Data (lines 1-50)
- `RAW_MOJIMATCH` — 13 sessions, ~6 unique users. Fields: uuid, createdAt, graderLabels[], playerLabels[], coins, sessionTime
- `RAW_CHARADES` — 15 user profiles. Fields: uuid, sessions, avgPoints, decks[], firstPlay, lastPlay, completedGames
- `RAW_SOUND` — 1 user. Fields: uuid, sessions, decks[], firstPlay, lastPlay, totalPoints
- `DECK_COLORS` — hex map for ~19 deck names. Helper: `dc(deckName)` returns color or fallback `#64748b`

### Profile Builder (lines 52-69)
`buildProfiles()` merges the three data arrays into a single `{ [uuid]: profile }` object.

Profile shape:
```js
{ uuid, charades: {...} | null, mojiMatch: [{ ...session, accuracy }], soundChallenge: {...} | null }
```
MojiMatch accuracy is computed here: element-wise comparison of `graderLabels` vs `playerLabels`.

### Recommendation Engine (lines 72-160)
`getRecs(profile)` → sorted array of `{ pri, icon, title, msg, color, tag }`.

Computed metrics:
- `totalSess` — sum of all game sessions
- `avgAcc` — mean MojiMatch accuracy (null if no sessions)
- `mojiTrend` — last session accuracy minus first (null if <2 sessions)
- `daysSince` — days since last play across all games (reference: `2026-02-27` hardcoded)
- `compRate` — charades completion percentage

10 rules, evaluated in order. Output sorted by priority (HIGH=0 > MEDIUM=1 > LOW=2).

Key thresholds: inactive >60d, drop-off >14d, gap >5d, low completion <50%, accuracy needs-support <50%, growing <75%, strong >=75%, trend declining <-15pp, improving >+10pp, low scores <1.5 avg, single deck + >3 sessions, power user >=25 sessions.

Full algorithm documented in `DiggyPop_Recommendation_Algorithm.pdf`.

### UI Components (lines 162-218)
- `Stat` — metric card (label, value, sub, color, icon)
- `Ring` — SVG donut chart for accuracy (green >=80%, amber >=60%, red <60%)
- `Rec` — recommendation card with priority badge + tag
- `Chip` — colored deck name badge

### Main App Component (lines 220-512)

**State:**
- `sel` — selected user UUID or null
- `search` — user search input text
- `tab` — `"overview"` or `"users"`

**Memoized values:**
- `profiles` — merged user profiles (built once)
- `users` — profiles sorted by total sessions descending
- `filtered` — users filtered by search text
- `agg` — aggregate stats: total users, sessions, active (30d), avg accuracy, unique decks
- `deckPop` — deck popularity sorted descending
- `gd` — game distribution (charades/mojimatch/sound session counts)
- `attention` — top 8 users with HIGH priority recs

**UI Layout:**
- **Header** (dark, `#111827`): logo + title, algorithm docs link, overview/users tab switcher
- **Overview tab**: 5 stat cards → game distribution + deck popularity → attention list → all users table
- **Users tab**: left sidebar (sticky, searchable user list) + right detail panel (stats, decks, mojimatch history, recommendations)

## Styling Conventions

- All inline styles (no CSS modules, no Tailwind, no styled-components)
- Font stack: DM Sans (body), Space Grotesk (headings/numbers) — loaded via Google Fonts `<link>` in JSX
- Color palette: dark bg `#111827`/`#1F2937`, light bg `#f6f7fb`/`#fff`, borders `#e8ecf1`, text `#1e293b`/`#475569`/`#94a3b8`
- Priority colors: HIGH `#EF4444`/`#FEE2E2`, MEDIUM `#F59E0B`/`#FEF3C7`, LOW `#10B981`/`#DCFCE7`
- Border radius: 8-16px, box shadows minimal (`0 1px 3px rgba(0,0,0,0.04)`)
- Max width: 1200px centered

## Important Notes

- The reference date for "days since last play" is **hardcoded to `2026-02-27`** — not `Date.now()`
- No external API calls — everything is rendered from the three `RAW_*` arrays
- The `getRecs()` function is called per-user on demand (in the users table it's called for every row on every render)
- Clicking an attention row or table row sets `sel` and switches to the Users tab
- The overview/users tab switch clears selection when going back to overview

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sdoychev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sdoychev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
