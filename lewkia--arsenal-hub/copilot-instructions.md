## arsenal-hub

> A single-page Arsenal FC fan dashboard built by Polla.

# Arsenal Hub — Claude Project Context

## What this is
A single-page Arsenal FC fan dashboard built by Polla.
- **File**: everything lives in `index.html` (HTML + CSS + JS, no build step)
- **Deployed**: Vercel — run locally with `vercel dev`
- **Local dev**: `vercel dev` at `http://localhost:3000` (required for API rewrite to work)

## Architecture

### API — football-data.org (free tier)
- Arsenal team ID: `57`
- All API calls go through `apiFetch()` which routes via `/api/football/*`
- Vercel rewrites `/api/football/*` → `https://api.football-data.org/v4/*` (see `vercel.json`)
- This bypasses CORS — do NOT call the API directly from the browser
- When opened as `file://` (no server), falls back to `corsproxy.io` as CORS proxy
- **Free tier limitations**: no form data in standings, limited squad fields — work around these, don't fight them

### News — rss2json proxy
- Fetches 4 RSS feeds via `rssFetch()` which calls `api.rss2json.com`
- Sources: Football.London, Arseblog, Just Arsenal, Daily Cannon

## Section order (nav + DOM)
1. Home (`#s-next-match`) — next fixture + countdown
2. Table (`#s-standings`) — EPL standings
3. Fixtures (`#s-fixtures`) — upcoming 5 matches
4. Results (`#s-results`) — last 5 results
5. Squad (`#s-squad`) — static data, NOT from API
6. Scorers (`#s-scorers`) — top scorers (EPL + Arsenal)
7. (no nav link) Assists (`#s-assists`) — top assists, follows scorers
8. News (`#s-news`) — news feed

## Squad section
- **Source**: `players.txt` (manually maintained, do not use API for squad)
- Squad data is hardcoded in `loadSquad()` as a static JS object
- `players.txt` is the source of truth — sync JS when it changes
- Player cards: tap to expand, show age / contract / market value
- Status badges: New Signing (green), Loan Out (amber), Loan In (blue), Captain (red)

## CSS conventions
- CSS variables defined in `:root` — always use these, never hardcode colours
  - `--red: #EF0107`, `--bg-card: #16213e`, `--bg: #1a1a2e`, `--secondary: #aaaaaa`
- `hide-sm` class = desktop only (hidden on mobile via media query)
- Mobile breakpoint: `768px`
- Responsive grid: `auto-fill, minmax(...)` pattern throughout

## Known decisions / don't redo these
- **Form column removed** from standings — `row.form` is null on the free API tier
- **League table mobile**: shows P, GD, Pts. Desktop adds W, D, L columns
- **Squad not from API**: API returns incomplete/inconsistent data — squad is always static from `players.txt`
- **Chart.js removed**: was used for season stats (Section 12, now scrapped)
- **Sections 10–13 removed**: Injury Tracker, H2H, Season Stats, On This Day — scrapped for simplicity

## Key files
| File | Purpose |
|------|---------|
| `index.html` | Entire app |
| `vercel.json` | API rewrite + output directory config |
| `players.txt` | Squad source of truth (manually updated) |
| `package.json` | Minimal — only `@vercel/analytics` dep + dummy build script |
| `.env.local` | API key (`X-Auth-Token`) — not committed |

## API key
Stored in `.env.local` as an env var, injected at runtime. Do not hardcode it.
The `apiFetch()` function reads it from the page — check how it's referenced before changing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lewkia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
