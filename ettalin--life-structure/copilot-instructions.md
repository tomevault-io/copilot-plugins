## life-structure

> Orientation for Claude Code working in this repo.

# CLAUDE.md

Orientation for Claude Code working in this repo.

## What this is

`life-structure` is a single-file progressive web app — a quiet personal system for money, routines, wellness, and supplements. Its purpose is to reduce daily decision fatigue and keep working when the user is stressed, tired, or emotional. Design intent: calm, low-stimulation, non-judgmental. The tone of any copy you write should match: gentle, plain, never shaming.

## Architecture

Almost everything lives in **`life-structure.html`** — HTML, CSS, and JavaScript in one file, no framework, no build step, no external runtime dependencies.

- **CSS** is in a single `<style>` block driven by CSS custom properties in `:root`. Theme is dark with gold (`--gold`), green (`--green`), and blue (`--blue`) accents. Reuse the variables; do not hardcode colors.
- **JS** is one inline `<script>` at the end of `<body>`. It is plain ES (no modules, no bundler).

Supporting files make it an installable PWA:
- `manifest.json` — metadata + icon references.
- `service-worker.js` — network-first cache with offline fallback. Bump `CACHE` when assets change.
- `icons/` — generated PNGs (192, 512, 512-maskable).

## State model

A single object `state` is persisted to `localStorage` under `STORE_KEY = "lifeStructure.v1"`.

```
state = {
  finance:    { income, bills, rate, efTarget, savings, treatLimit },
  treats:     { spent, weekStamp, log:[amt...] },     // log powers Undo
  anchors:    { stamp, done:[bool x4] },
  supps:      { stamp, done:[bool x4] },
  fitness:    { stamp, done:[bool x3] },              // Stretching / Calisthenic strength / Posture (FITNESS array)
  planner:    { Monday:{p,m,w}, ... },                // p=priority, m=money move, w=wellness move
  spendTimer: { endsAt },                             // ms epoch; persists the 20-min wait across reloads
  streaks:    { anchors:{n,last}, supps:{n,last}, fitness:{n,last} }, // consecutive-day completion
  xp:         0,                                      // System XP (see RANKS / levelInfo)
  awards:     { stamp, anchors:[bool x4], supps:[bool x4], fitness:[bool x3] }, // per-day XP-award guard
  invest:     { holdings:[ {id,ticker,shares,cost,price,market,currency} ], history:[ {d,v} ], updated, fx },
                                                             // market: US|JSE; currency: USD|JMD; fx = JMD per 1 USD; history in baseCcy
  property:   { target, saved, monthly, log:[amt...] },      // property fund goal + Undo
  certs:      [ {id,name,target,progress,status,awarded} ],  // status: Planned|In progress|Earned
  baseCcy:    "JMD",                                         // base currency for savings/property/net worth (JMD|USD)
  vitals:     [ {id,d,sys,dia,weight,hr,steps} ],            // BP / weight / resting HR / steps; one record per date (merged)
  view:       "full",                                        // "full" | "minimal" (the Now focus screen)
  schedule:   { supps:{i:"HH:MM"}, fitness:{i:"HH:MM"} },    // time-of-day overrides; defaults live on supplement/FITNESS `.time`
  suppList:   [ {name,dose,schedule,purpose,tags,time,notes} ], // user's own supplements; empty → generic SUPPLEMENTS defaults
  profile:    { considerations:[], allergies:[], avoid:[] }  // personal health profile; ships empty
}

**Privacy rule:** the committed code must stay generic. Personal supplements and health-profile content live only in `state` (`suppList`, `profile`) — never hardcode them back into HTML/JS/docs. `SUPS()` resolves `state.suppList` → generic `SUPPLEMENTS` fallback; always use it (not the constant) when reading the active list. `*.private.json` is gitignored for local personal-data backups.
```

Bump `STORE_KEY` only for breaking schema changes (wipes user data). New keys are merged in `load()` with defaults, so additive changes are safe without a bump.

Key functions:
- `load()` / `save(quiet)` — read/write `localStorage`; `save()` shows a brief "Saved" flash unless `quiet`. `load()` deep-merges defaults for each nested slice so old saves stay valid.
- `applyResets()` — clears daily items (anchors, supps, **awards**) when `stamp !== today()` and treats when `weekStamp !== weekStamp()`. Called on load and on `visibilitychange`.
- `compute()` — money math: `afterBills = income - bills`; `save = afterBills * rate/100`; `flex = afterBills - save`.
- **System:** `xpForLevel(L)`, `levelInfo(xp)` (→ level/into/span/pct/nextAt), `rankFor(L)` (RANKS E→S), `awardXP(n)` (detects level-ups → `showLevelUp`), `renderSystem()`.
- `render*()` — one per UI area: `renderDashboard`, `renderToday`, `renderAnchors`, `renderSupps`, `renderPlanner`, `renderGlance`, `renderLabs`, `renderSystem`, `renderInvest`, `renderProperty`, `renderCerts`, `renderAscSummary`.
- Ascension actions: `addHolding`/`delHolding`/`updHoldingPrice`, `addProperty`/`undoProperty`/`addSuggestedToProperty`/`usePlanAsMonthly`, `addCert`/`updCert`/`earnCert`/`removeCert`. Forms bound in `bindFinance()` and `bindAscension()`.
- `renderNetWorth()` = savings/EF + property fund + portfolio value (Dashboard card). `snapshotPortfolio()` records one portfolio data point per day → `renderSpark()` draws an inline SVG sparkline.
- `fetchPrices()` — **on-demand, keyless** live quotes. Routes by `holding.market`: **US** via `fetchQuote()` (`https://r.jina.ai/` → Yahoo `chart` endpoint, parses `regularMarketPrice`); **JSE** via `fetchJSEPrices()` (jina → `jamstockex.com/trading/trade-summary/`, regex-parses the markdown table into `{SYMBOL: close}`). Also fetches `USDJMD=X` to auto-update `invest.fx`. Only runs on tap. The service worker deliberately ignores cross-origin requests so these calls hit the real network instead of the offline HTML fallback — **keep that guard.**
- **Currency truth:** `fmtC(n,ccy)` (J$ vs $), `ccyOf(h)`, `toBase(amt,ccy,base,fx)` (returns `null` if the rate is missing — never fakes a conversion), `portfolioBaseValue()`. The portfolio summary shows **per-currency subtotals** (never a blended JMD+USD number); net worth converts to `baseCcy` and warns if a needed rate is missing. JSE prices are JMD, US prices USD.
- `renderFitness()` — daily 3-item checklist (FITNESS array) in the Health tab; same XP/streak/award pattern as anchors/supps (+10 XP each).
- **Fitness plans:** `PLANS` (6 curated bodyweight routines: stretch, posture, strength Foundation/Builder, **Asian-style low-impact cardio**, **lymphatic drainage**; each routine's `fitnessIdx` maps to a daily Fitness item). FITNESS has 5 items now — `load()` normalizes `fitness.done` / `awards.fitness` to `FITNESS.length`, so growing the list never corrupts old saves. `renderPlans()` builds the accordions; `markFitness(idx)` logs a routine → checks the daily item + XP; `refreshPlanButtons()` keeps the "Logged today" buttons in sync.
- **Vitals:** `renderVitals()`, `addVital()`, `delVital()`, `bpCategory(sys,dia)` (Normal→Crisis, informational only). Trends drawn with the reusable `sparkSVG(values,{color,label,h})` helper (also used by the portfolio sparkline).
- **Backup:** `exportData()` downloads the whole `state` as JSON; `importData(input)` validates + restores it (then re-runs `load()`/`applyResets()`/`renderAll()`/binds). Lives in the Health footer.
- **Now mode (minimal view):** `setView("full"|"minimal")` toggles `body.minimal-on` (CSS hides header/`.wrap`/nav and shows `#minimal`); `applyView()` is the single entry point (called from init, hardReset, import). `renderMinimal()` shows only timed tasks due now (meds + fitness whose `schedTime` ≤ now and not done) as big tap-to-complete cards, plus a muted "Later today" list and an "all caught up / next at…" empty state; a 30s interval refreshes the clock + due list while active. `toggleTask(kind,i)` marks done + awards XP, shared with the full view's state.
- **Schedule:** each SUPPLEMENT/FITNESS has a default `.time` ("HH:MM"); `state.schedule` holds per-item overrides. `schedTime(kind,i)` resolves override→default. `renderSchedule()` builds the editable time inputs in the Health "Daily Schedule" card. Helpers: `timeToMin`, `fmtTime`, `nowMin`, `timedTasks()`.
- **Calendar export:** `exportICS()` writes a `.ics` file — one daily-recurring VEVENT (with VALARM) per med/fitness item at its `schedTime`. There is **no live calendar web API** without OAuth+server, so this is a one-time import into Apple/Google/Samsung/Outlook; re-run after changing times.
- **Samsung Health import:** there is **no live web API** for Samsung Health / Galaxy Watch — a browser PWA can't background-sync. `importSamsung(input)` reads the CSV files from Samsung Health's "Download personal data" export; `parseSamsungCSV(text)` finds the header by **max distinct column tokens** (the metadata first line contains a single token like "weight" and must not win), detects systolic/diastolic/pulse/weight/heart_rate/step columns + a time column (`toStamp` handles `YYYY-MM-DD HH:MM:SS` and epoch ms), and `finishSamsung()` merges records into `state.vitals` by date (steps kept as the day's max). If a real export won't parse, get a sample and tune `parseSamsungCSV`.
- Content arrays near the top: `SUPPLEMENTS`, `LABS`, `ANCHORS`, `DAYS`, `RANKS`, `XP`.

## System FX / animations

The *Solo Leveling* feel is carried by motion, all of it disabled under `prefers-reduced-motion` (the global reduce block neutralizes durations; `popXP()` also early-returns via `reduceMotion()`):
- **Level-up window** (`#levelup` → `.lu-panel`): full-screen dimmed/blurred backdrop with a centered glowing panel (corner brackets, scanline sweep, flicker-in "LEVEL UP"). `showLevelUp(L)` shows it ~2.3s; `awardXP` triggers it + `navigator.vibrate`.
- **Floating +XP** (`popXP(n)`): spawns a transient `.xp-pop` that rises and fades on every XP gain.
- **XP-bar pulse**: `renderSystem` adds `.pulse` to `#xp-bar` when `state.xp` increased since last render (`_lastXp`).
- **Ambient**: rank-badge glow pulse; `.sys-scan` holographic scanline drifting down both `.sys-card`s.
- **Entrances**: `section.view.active > .card` and `#minimal .m-task` fade/slide-up with nth-child stagger on every view switch.

Keep new motion transform/opacity-only and ensure the reduce path stays clean.

## XP / gamification

Subtle "System" layer inspired by *Solo Leveling*. Completing a daily anchor (+12), supplement (+6), or fitness item (+10) awards XP **once per day** (guarded by `state.awards`; un-checking never removes XP). Property contributions give small XP; earning a certification gives +200. Level + Rank (E→S) derive from total XP. Keep it gentle and non-shaming — XP only ever goes up, there are no penalties.

## Tabs

Five bottom-nav views: **Dashboard** (System status + Today + Net Worth + money + anchors), **Planner**, **Finance**, **Ascension** (investments incl. JSE, property fund, certifications, currency settings), **Health** (Meds & Supplements checklist + notes, **Fitness** checklist + curated **Fitness Plans**, **Vitals** BP/weight tracker, profile, labs, timeline, **data backup** export/import). Bottom nav is capped at 5 for thumb reach. The **Today** snapshot has 4 chips (Anchors, Supplements, Fitness, Treat).

`renderAll()` re-renders everything; call it after mutating `state`.

## Conventions

- **If you change `state`, call `save()` then the relevant `render*()`.**
- Edit the supplement/health/labs content by editing the data arrays, not the markup.
- Bump `STORE_KEY` only for breaking schema changes (it will wipe users' data — avoid).
- Bump `CACHE` in `service-worker.js` whenever you change cached files, or installed clients will keep stale assets.
- Keep it dependency-free and single-file unless the user explicitly asks to split into modules.

## How to run / verify

```bash
python3 -m http.server 8000     # then open http://localhost:8000/life-structure.html
```

Quick logic check without a browser: extract the first `<script>` block and eval it under jsdom-style mocks (income 4000 / bills 1500 / rate 20 should give save 500, flex 2000; a stale week stamp should reset `treats.spent` to 0).

## Health/medical content

The supplement, dosing, labs, and timeline content is personal and specific. Do not invent or alter medical figures. Preserve the "tracking only, not medical advice" framing wherever it appears.

---
> Source: [ettalin/life-structure](https://github.com/ettalin/life-structure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
