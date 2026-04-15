## stablemate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

StableMate is a barn/equestrian management SPA for Oasis Farm. It handles rider bookings, horse schedules, and multi-role access (Rider, Parent, Staff). Built with vanilla JavaScript, Supabase (PostgreSQL + Auth), and deployed on Vercel.

## Commands

### Main test suite (AFB/HNR feature tests)
```bash
npm test                        # Run all Playwright tests
npm run test:afb                # AFB tests only
npm run test:hnr                # HNR tests only
npm run test:cross              # Cross-feature tests only
npm run test:headed             # Run with visible browser
npm run test:ui                 # Interactive Playwright UI
```

### Regression suite (stubs Supabase, uses fixture data — no live DB needed)
```bash
cd stablemate-regression
npm install && npx playwright install
npm test
```

### Deploy
```bash
./deploy.sh /path/to/index.html
```

No build step — the app is plain HTML/CSS/JS. The regression suite stubs Supabase in-browser using fixture data from `stablemate-regression/fixtures/mockData.js`.

## Architecture

### Entry point and script load order

`index.html` is the entire app. Scripts must load in this order (order matters — globals are set up sequentially):

```
config.js       → globals: sb, BARN_NAME, horses[], riders[], bookings[], currentUser, etc.
init.js         → calls initAuth() on DOMContentLoaded
auth.js         → Supabase Auth, user_profiles lookup, sets currentRole
data.js         → loadAll() fetches all Supabase tables into global arrays
permissions.js  → role-based access helpers
utils.js        → misc helpers
crud.js         → booking CRUD + conflict detection (uses globals set above)
ui-shared.js    → calendar, modals, toasts
afb-data.js     → AFB/HNR data layer (must load BEFORE afb-ui.js)
afb-ui.js       → AFB/HNR sheet UI
ui-rider.js     → rider screen
ui-parent.js    → parent screen
ui-staff.js     → staff dashboard
ui-browse.js    → horse browsing
sheets.js       → booking form overlays
```

### Global state (set in config.js, mutated throughout)

```js
let horses=[], owners=[], riders=[], bookings=[], schedules=[], shows=[], lungeRequests=[];
let afbEntries=[], hnrEntries=[];
let currentUser=null, currentRole='rider', currentChildId=null;
```

All modules read from and write to these globals directly — there is no state management library.

### Data flow

1. `initAuth()` checks for a Supabase session
2. On session found: loads `user_profiles` → sets `currentUser`, `currentRole`
3. `loadAll()` fetches all tables into the global arrays
4. The appropriate role screen is rendered: `showRiderScreen()`, `showParentScreen()`, or `showStaffScreen()`

### Supabase tables

| Table | Purpose |
|---|---|
| `auth.users` | Supabase native auth |
| `user_profiles` | Links auth to app role (rider/parent/staff) |
| `riders` | Rider profiles with `approved_horses[]`, `parents[]` |
| `horses` | Horse profiles with `access` level |
| `bookings` | Core booking records |
| `away_from_barn` | Rider unavailability (AFB) |
| `horse_unavailable` | Horse unavailability (HNR) |
| `shows` | Competition shows; adding a rider auto-creates an AFB entry via `syncShowAfb()` |
| `lunge_requests` | Lunging requests |
| `trainer_schedules` | Weekly trainer availability bands |

### AFB/HNR integration

`checkBookingConflicts(riderId, horseId, dateStr, timeStr)` must be called before saving any booking. It returns `{warnings, canProceed}`. Live warnings are wired to date/rider/horse/time field changes via `updateBookingConflictWarnings('b')` (staff) or `updateBookingConflictWarnings('rb')` (rider/parent).

When saving a show, call `syncShowAfb(savedShow)` to auto-create/remove AFB entries for riders on that show.

### Booking types (defined in `typeConfig` in config.js)

`lesson`, `arena-supervised`, `arena-independent`, `vet`, `farrier`, `competition`, `lunge`, `turnout`, `walker`

### Roles

| Role | Can do |
|---|---|
| `rider` | View/manage own bookings and AFB |
| `parent` | Same as rider, scoped to their children via `currentChildId` |
| `staff` | Full CRUD across all riders, horses, bookings, shows, AFB, HNR |

Permission checks live in `permissions.js`; enforcement is in `crud.js`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henrikamren)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/henrikamren)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
