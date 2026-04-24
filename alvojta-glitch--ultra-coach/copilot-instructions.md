## ultra-coach

> This file is auto-loaded in future Claude Code sessions. It captures everything needed to safely maintain this project without losing user data.

# Ultra Coach — Project Context for Claude

This file is auto-loaded in future Claude Code sessions. It captures everything needed to safely maintain this project without losing user data.

## Project overview

- **Single-file React PWA** for ultra-running coaches and athletes
- Entire app lives in `index.html` (~4100 lines, hand-written + Babel-transpiled `React.createElement` calls)
- No build step, no package.json, no bundler — edit and push
- CDN deps: React 18, ReactDOM 18, Tailwind, Lucide-React 0.383.0, Outfit font
- **Live URL:** https://alvojta-glitch.github.io/ultra-coach/
- **Repo:** https://github.com/alvojta-glitch/ultra-coach
- **Local working dir:** `C:\Users\alvoj\Downloads\ultra-coach\`
- **Git identity:** `alvojta-glitch` / `alvojta@gmail.com` (configured locally in this repo)
- The OTHER `index.html` at `C:\Users\alvoj\Downloads\index.html` is a STALE DUPLICATE — ignore it, only edit the one inside `ultra-coach/`

## Architecture summary

| Screen | Function |
|---|---|
| `LoadingScreen` | Boots, checks cloud creds, routes to home/onboarding/cloudsetup |
| `CloudSetupScreen` | First-run JSONBin setup (optional, can skip) |
| `OnboardingScreen` | 5-step wizard → builds `profile` + generates Week 1 plan via Claude API (or fallback) |
| `HomeScreen` | Greeting, API key warning, race countdown, 7-day fatigue, today's workout, action buttons |
| `PlanScreen` | Week view with workout details, mark complete/skip/undo, generate next week (with confirmation) |
| `QuickScreen` | AI-generated short mobility/strength/recovery sessions |
| `LogScreen` | RPE + distance/duration/elevation/terrain logging, returns coach feedback |
| `ChatScreen` | Persistent chat with the coach, context-aware, clearable |
| `SettingsScreen` | Four tabs: Profile, API (Anthropic key), Data (cloud + export/import), Danger (reset) |

## Data persistence — CRITICAL FACTS

### localStorage keys (NEVER rename without migration)

```js
const KEYS = {
  PROFILE: "ultra-coach:profile",       // Profile object
  PLAN: "ultra-coach:plan",             // Current week plan
  LOG: "ultra-coach:log",               // Array of logged workouts (capped at MAX_LOG_ENTRIES=400)
  WEEKS: "ultra-coach:weeks",           // Map of all stored weeks { "week-1": {...}, ... }
  JBIN_KEY: "ultra-coach:jbin-key",     // JSONBin master key (cloud sync)
  JBIN_BIN: "ultra-coach:jbin-bin",     // JSONBin bin ID
  ANTHROPIC_KEY: "ultra-coach:anthropic-key", // User's Anthropic API key (device-local, NEVER synced)
  CHAT: "ultra-coach:chat"              // Persisted chat history (capped at last 50 msgs)
};
```

### Guarantees

1. Pushing new code to the same origin (`alvojta-glitch.github.io`) does **NOT** wipe localStorage
2. Browser hard-refreshes do NOT wipe localStorage
3. Changing a `KEYS.*` string value DOES orphan old data — avoid or write a migration
4. Changing the data shape (renaming fields) requires a migration in `loadAll` or on first read
5. Clearing browser site data wipes everything — user must export first
6. iOS Safari can purge localStorage after ~7 days of no visits — PWA install or JSONBin sync mitigates

### Backup paths available to the user

- **JSONBin sync** (Settings → Data): auto-syncs profile/plan/log/weeks to a private bin on jsonbin.io. Master Key + Bin ID can restore on any device. `ANTHROPIC_KEY` and `CHAT` are intentionally NOT synced.
- **Export/Import** (Settings → Data): copies `{profile, plan, log}` JSON to clipboard. Paste back to restore.

## Safe update workflow

```bash
cd C:/Users/alvoj/Downloads/ultra-coach
# edit index.html
git add index.html
git commit -m "describe change"
git push
```

Wait ~60s, hard-refresh (Ctrl+Shift+R). Pages auto-deploys on push.

## Schema migration pattern

When adding/renaming fields, handle old records gracefully:

```js
// In loadAll() after sGet(KEYS.PROFILE):
const p = await sGet(KEYS.PROFILE);
if (p) {
  let migrated = false;
  // Example: normalize old string timeline to number
  if (typeof p.timeline === "string") {
    p.timeline = parseInt(p.timeline) || 0;
    migrated = true;
  }
  // Example: add default for new field
  if (p.targetVert === undefined) {
    p.targetVert = 0;
    migrated = true;
  }
  if (migrated) await sSet(KEYS.PROFILE, p);
  setProfile(p);
}
```

Run migrations idempotently — they should be no-ops on already-migrated data.

## Already-fixed bugs / gotchas (don't regress)

1. **`window._Lucide` fix**: The original file created `var LucideReact` but destructured from `window._Lucide`, causing a blank black screen. The current normalization IIFE assigns `window._Lucide` and provides a Proxy stub fallback if the CDN fails. Do not remove the IIFE.
2. **Anthropic API calls**: Must include `x-api-key`, `anthropic-version: 2023-06-01`, and `anthropic-dangerous-direct-browser-access: true` headers. Model is `claude-sonnet-4-5`.
3. **API key is device-local**: Stored in `KEYS.ANTHROPIC_KEY`, read by `callCoach` on every call, NEVER included in JSONBin sync records. Each device enters its own key. Do not change this.
4. **`timeline` is a number (weeks)**: Old versions stored it as `"16 weeks"`. Code uses `typeof === "number"` guards when reading for backward compat. Any new code should treat it as a number.
5. **`coachResponse` is NOT persisted in log entries**: Shown once from local state, then discarded. Keeps localStorage lean. Don't reintroduce.
6. **Log entries capped at `MAX_LOG_ENTRIES = 400`** via `.slice(-MAX_LOG_ENTRIES)` on write.
7. **Chat history capped at 50 messages** on write to `KEYS.CHAT`.
8. **Workout state has two booleans**: `completed` and `skipped`. They are mutually exclusive. Never set both true. Undo sets both false.
9. **Plan week advancement requires confirmation** (`confirmGen` state in `PlanScreen`). Don't remove the gate — user specifically asked for it.

## What NOT to do

- Don't `git add .` or `git add -A` from `C:\Users\alvoj\Downloads\` (parent dir is full of unrelated files). Always work inside `ultra-coach/`.
- Don't commit any `sk-ant-...` API key, JSONBin master key, or `.env` file.
- Don't create a `package.json` / build step unless the user explicitly asks for it. The single-file design is intentional.
- Don't remove the fallback `makeFallbackPlan` — it's the safety net when Claude API is unavailable.
- Don't change the GitHub Pages URL / repo name without a data migration plan — it changes the origin and orphans all existing localStorage.
- Don't rename `KEYS.*` values.

## Future feature ideas (not yet implemented)

- Weekly vert goal in onboarding (currently only per-workout elevation tracking)
- TSS/CTL/ATL fatigue modeling (currently simple 7-day RPE average)
- Nutrition/fueling plan generator
- Exercise progression tracking for strength sessions
- Weather/terrain-aware coaching prompts
- Export as `.fit` / `.gpx` for import into Strava/Garmin

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alvojta-glitch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
