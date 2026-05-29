## armchairgms

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this app is

ArmchairGMs is a dynasty fantasy football analytics tool. Users connect their Sleeper league ID to get roster breakdowns, trade analysis, rankings, rookie scouting, and more. It's a single-page app served by a minimal Flask backend.

## Running locally

```bash
# Activate virtualenv (Windows)
.venv\Scripts\activate

# Run dev server
flask run --debug
# or
python app.py
```

The app runs at `http://127.0.0.1:5000`. `FLASK_DEBUG` env var controls debug mode.

## Deployment

Deployed on Render via `Procfile`:
```
web: gunicorn app:app
```

The `ADMIN_KEY` environment variable must be set on Render to protect the `PUT /api/players` endpoint.

## Architecture

### Backend (`app.py`)
Flask does almost nothing — it serves `templates/index.html` (injecting the full player dataset via Jinja2) and exposes two API routes:
- `GET /api/players` — returns the player/pick JSON
- `PUT /api/players` — updates `data/players.json` (requires `X-Admin-Key` header matching `ADMIN_KEY` env var)

After the initial page load, everything is client-side. Flask is not involved in auth, league data, or trade analysis.

### Player data (`data/players.json`)
Compact format to minimize payload. Short field names:
- `n` = name, `p` = position, `t` = team, `a` = age, `s` = score, `h` = hardcoded (top-tier lock), `T` = type (`"player"` or `"pick"`), `r` = rank
- Picks also have: `y` = year, `ro` = round, `sl` = slot (Early/Mid/Late)

This is expanded to full names in `static/js/auth.js` at runtime into the global `DATA` array.

### Frontend JS files (`static/js/`)

The JS is split by feature. Load order matters — `auth.js` must come first because it defines `DATA` and `CU` (current user) that everything else depends on.

| File | Responsibility |
|---|---|
| `auth.js` | Data initialization, Supabase auth (signup/login/logout/password reset), session persistence, `launch()` entrypoint |
| `nav.js` | Multi-league dropdown, page navigation (`showPg()`), avatar rendering, name normalization |
| `core.js` | Global state (`MODE`, `SEL`, `GP`, `RP`, `SCORING`, etc.), scoring logic (`getScore()`), tier labels, Flock rankings parser |
| `rankings.js` | Rankings table, player card (`renderCard()`), trade button actions, league overview rendering, roster blurbs |
| `sleeper.js` | Roster modal (per-team player list with picks) |
| `trade.js` | Trade calculator UI (`renderTA()`, trade tags) |
| `trades.js` | Recent trades feed from Sleeper |
| `myteam.js` | "My Team" tab — user's own roster view |
| `startsit.js` | Start/Sit comparison tool |
| `rookies.js` | Rookie rankings tab |
| `movers.js` | Value movers (uses localStorage snapshot for delta) |
| `targets.js` | Trade target suggestions |
| `history.js` | League history / championship tracker |
| `roster.js` | Roster analysis helpers |
| `draftgrade.js` | Draft grade tool |
| `trade-search.js` | Search within the trade calculator |

### Global state (defined in `core.js`)
- `DATA` — full array of all players + picks, sourced from `RAW` (injected by Flask)
- `CU` — current user object `{em, lid, token, uid}`
- `GP` / `RP` / `RP2` — give/receive/receive-2 arrays for the trade calculator
- `SEL` — currently selected player
- `SCORING` — `'SF'` (Superflex) or `'1QB'`
- `TRADE_TYPE` — `2` or `3` (2-way vs 3-way trade)
- `MODAL_DATA` — league roster data after a Sleeper sync
- `LEAGUES` — array of saved league objects `{id, name}`

### Auth & user data (Supabase)
`auth.js` calls Supabase REST directly from the browser using a hardcoded public anon key (this is the normal Supabase pattern — the anon key is safe to expose). User profiles are stored in a `profiles` table with columns `id`, `email`, `league_id`, `scoring`, `leagues` (JSONB).

Session is persisted to `localStorage` under `agm_s`. League list is persisted under `agm_leagues_<uid>`.

### Scoring logic
`getScore(d)` in `core.js` applies a `QB_MULTS` positional adjustment when in 1QB mode and a rookie age boost (up to +12%) for players age ≤23 ranked ≥22. The `hardcoded` flag on top-ranked players (`h: true`) protects them from the Flock rankings parser overwriting their scores.

### Sleeper integration
League data is fetched client-side from the Sleeper public API (no key required). `loadLeague()` in one of the JS files hits `https://api.sleeper.app/v1/league/<id>` and related endpoints to pull rosters, matchups, and trades.

---
> Source: [kevindougherty1/armchairGMs](https://github.com/kevindougherty1/armchairGMs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
