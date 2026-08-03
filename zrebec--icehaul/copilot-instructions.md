## icehaul

> Guidance for Claude Code when working in this repository.

# CLAUDE.md — Ice Haul

Guidance for Claude Code when working in this repository.

## What this is

**Ice Haul** is a ZX-Spectrum-flavoured micro-simulator of ice-road trucking. Not ETS2 — its ZX hallucination. The fantasy is *risk management*, not speed: tyre pressure, cargo balance, ice patches, wind, fuel, driver fatigue. Every metre is a small decision.

GitHub repo: `zrebec/ice-haul`. Built on **[zx-kit](https://github.com/zrebec/zx-kit)** (npm `zx-kit@^0.21.0`).

## Canvas & resolution

- **Game pixels: 256 × 192** (32 × 24 cells at `CELL=8`). Pure ZX Spectrum native.
- Initialise with `setupCanvas(canvas, 4, 256, 192)` → 1024 × 768 CSS px.
- Integer scaling only. Never alter `CELL` or the palette in `zx-kit/src/palette.ts`.
- **Colour clash is a feature, not a bug.** Use `drawBitmapAttrs` with `AttrMap` (per 8×8 cell ink/paper) so two adjacent palette colours visibly bleed across a sprite — that's the look.
- Palette is the 15 hex values in `C` from `zx-kit/src/palette.ts`. No other colours, ever.

## HUD layout target (matches `first_impression.png`)

```
┌────────────────────────────────┐ rows 0–3   (status bar, 4 cells / 32 px)
│SCORE 002350       DIST 12.3km │
│                               │
│TIME 02:47       ICE AHEAD     │
│                               │
├────────────────────────────────┤
│                                │ rows 4–14  (drive viewport, 11 cells / 88 px)
│     <pseudo-3D road scene>     │   sky ~25%, road ~75%
│                                │
├────────────────────────────────┤
│E▮▮▮▮F   │    ◯ RPM    │ FREE  │ rows 15–23 (HUD, 9 cells / 72 px)
│RPM ▮▮▮▮▮│   RPM 0850  │ DRIVE │ 3 equal panels: drivetrain | tach+speed | mission
│GEAR 3/5 │   SPD  47    │       │
│GRIP ▮▮▮▮│             │       │
└────────────────────────────────┘
   85 px      85 px     86 px
```

HUD has **3 equal-width panels** (256/3). Left ("drivetrain") panel, top→bottom: **FUEL** bar, **RPM** bar, **GEAR** (current/total), **GRIP** bar — with short labels. Centre ("tachometer") panel: an **rpm dial** (needle = real engine revs, reddens at redline) plus a numeric **RPM** (real revs) and numeric **SPD** (km/h). Right: mission info placeholder. Status bar is **4 rows** — no "ICE TRUCKER" title.

> The centre panel was the **speed dial** until the lugging rework — it became a **tachometer** (owner idea 2.5, the "tach dial + prominent speed number" middle ground) so you can read real revs and see when you're lugging a too-tall gear. The left **RPM bar** now shows the raw ratio and CAN drop to 0 bars when lugging.

| Widget | zx-kit function |
|--------|-----------------|
| TACH | `drawDial` value = real rpm (`rpm × RPM_DISPLAY_REDLINE`), `min 0 max 2600`, `ticks: 5`. Needle `B_RED` at/above `REDLINE_RPM`, else `B_YELLOW`. Numeric RPM + SPD via `drawText`. |
| FUEL | `drawSegmentedBar` (horizontal). E in `B_RED`, F in `B_GREEN`, segments `B_YELLOW`. |
| RPM bar | `drawSegmentedBar` (horizontal, 7 seg). 3-stop gradient `[B_GREEN, B_YELLOW, B_RED]`; drops to 0 bars when lugging. |
| GEAR | `drawText` — current gear in `B_CYAN` (`B_RED` flash on a synchro-refused shift), `/N` total in `WHITE`. |
| GRIP | `drawSegmentedBar` (horizontal, single 6-seg bar). 3-stop gradient `[B_RED, B_YELLOW, B_GREEN]`. |
| Star field | Hand-plotted `STAR_POSITIONS` table (~17 points for compact sky). |
| Warning text `ICE AHEAD` | `drawTextCentered` + blink toggle in drive scene. |
| Kerb stripes | Pole-Position-style alternating `B_WHITE`/`B_YELLOW`, perspective-scaled width. |

## Gameplay model (decided)

- **Progression**: campaign + save. A series of seasonal routes, increasing difficulty, unlockable rigs/routes. Highscore table per route.
- **Decisions UX**: hybrid.
  - **Pit-stop scene** between segments — large decisions: tyre choice, cargo lashing, fuel fill, rest.
  - **Real-time in-drive** — small switches: heater, wipers, headlight high-beam, CB radio. Bound to number keys / gamepad shoulders. The risk is taking eyes off the road.
- **Audio**: AY engine drone modulated by RPM during drive + beeper SFX (warning beep, gear shift, ice crack, low-fuel chirp). AY tracker music for menus/intro/pit-stop only.

## Module layout (target)

```
src/
  main.ts            entry: setupCanvas, scene manager bootstrap, audio init on first input
  config.ts          LANGUAGE_CODE, debug flags, constants
  strings.ts         English UI strings (default locale)
  strings.sk.ts      Slovak (gitignored if needed; pickLocale via zx-kit i18n)
  save.ts            wires zx-kit save profile — campaign state, unlocks, highscores
  scenes/
    drive.ts         the main driving scene ✓ (phase 1)
    intro.ts         title + press-key                (phase ≥6)
    menu.ts          new game / continue / options    (phase ≥11)
    pitstop.ts       between-segment decisions        (phase 6)
    pause.ts         pushed on top of drive           (phase ≥6)
    gameover.ts                                       (phase 5)
  game/
    vehicle.ts       ✓ throttle/brake/steer + grip-scaled lateral physics
    road.ts          ✓ deterministic surface lookup (asphalt/ice)
  render/
    road3d.ts        ✓ scrolling pseudo-3D road with per-scanline surface
    truck.ts         ✓ rear-view player truck sprite
    hud.ts           ✓ bottom instrument cluster (SPEED wired, rest cosmetic)
    topbar.ts        ✓ score/title/dist/time/ice-ahead-blink
  audio/
    engine.ts        ✓ continuous square-wave drone pitch-modulated by speed
  game/              game-specific systems (do NOT push to zx-kit)
    vehicle.ts       traction, inertia, steering lag, brake distance
    road.ts          segment generator: curvature, slope, surface type
    weather.ts       wind vector, visibility, snow/rain emitter, ice patch placer
    cargo.ts         weight, centre of gravity, "shifting load" event
    driver.ts        fatigue curve, reaction time, day/night cycle
    risk.ts          random events (tyre blowout, frozen lock, blizzard window)
  render/            game-specific rendering on top of zx-kit primitives
    horizon.ts       sky gradient stripes + star field
    road3d.ts        pseudo-3D road: vanishing-point lines, side rails, dithered ice texture
    truck.ts         player sprite (low-poly silhouette ahead)
    hud.ts           composes dial/compass/bars/status from zx-kit ui.ts
    weatherFx.ts     snow particles, headlight cones (game-local — NOT zx-kit)
```

## What stays in IceRoads vs goes to zx-kit

**Stays here (game-specific):**
`vehicle.ts`, `weather.ts`, `road.ts`, `cargo.ts`, `driver.ts`, `risk.ts`, `weatherFx.ts`, `road3d.ts`, `truck.ts`, `horizon.ts`.

`weatherFx.ts` in particular: zx-kit's CLAUDE.md explicitly forbids a `particle.ts` module ("Spectrum philosophy: less is more"). We respect that — snow/rain stays game-local.

**Candidates to upstream to zx-kit later (only when a *second* zx-kit game would also benefit):**
- `drawHorizon` — sky-band gradient helper (generic for racing/space sims).
- `drawStarField` — twinkling random dot field with seed.
- `drawDashboardPanel` — wrapper around `drawBox` + `drawPanelTitle` in the recurring "title strip over framed box" pattern.

Do not preemptively upstream. One game is not a pattern.

## Engineering conventions

- **No new runtime deps.** Match zx-kit's `dependencies: {}` discipline. Vitest + TS + ESLint dev-only.
- **Singleton state in zx-kit modules** (`audio`, `ay`, `input`, `ui` bar registry) is fine for a single-canvas game. Don't try to multi-instance them.
- **TypeScript strict.** No `any`. All ink/paper params typed as `SpectrumColor`.
- **Game-pixel coords everywhere.** `setupCanvas` applies `ctx.scale(4, 4)` — never call `ctx.scale` again.
- **Frame loop**: fixed-`dt` ticks for physics (vehicle, weather), variable-`dt` for animation/render. Pass `dt` (ms) into every `tick*` function.
- **Scene stack** (`zx-kit/src/scene.ts`): drive scene is the long-running one; pitstop and pause `pushScene` on top so the underlying drive scene freezes via `onPause`.
- **Save** via `zx-kit/src/save.ts` profile. Bump `version` and write a `migrate` whenever the saved-state shape changes.
- **i18n** via `pickLocale` from `zx-kit/src/i18n.ts`. UI strings go through `L.STR_*`. English is the source-of-truth locale; Slovak follows.

## Build & run

```bash
npm install
npm run dev                                # vite dev server on localhost:5173
npm test                                   # vitest
npm run build                              # production bundle (tsc + vite)
node scripts/screenshot.mjs out.png        # headless capture of canvas bitmap (dev server must be running)
node scripts/drive-shot.mjs out.png 5      # boots, holds ArrowUp+ArrowRight for N seconds, then captures
```

**CI / Pages rule (2026-07-03):** the deploy workflow must use `actions/upload-pages-artifact@v5+`
and `actions/deploy-pages@v5+` — the Pages backend rejects v3-era artifacts since 2026-07-03 with a
generic *"Deployment failed, try again later"* (build/tests green, no error description). If a Pages
deploy fails like that, check these two action versions FIRST.

## Controls

- **ArrowUp** — throttle
- **ArrowDown** — brake
- **ArrowLeft / ArrowRight** — steer
- **D** — shift up · **A** — shift down (manual 5-speed gearbox)
- **ENTER (hold ~1.8 s)** — crank the engine to start / restart a stalled engine; releasing early resets the crank. `STARTING...` overlay while cranking (`CRANK_NEEDED_MS = 1800` in `config.ts`).

**The manual gearbox is core.** Each gear has a top speed — **1st caps at ~28 km/h, so you physically cannot reach 120 in a low gear** — and a torque band shown on the **RPM** gauge. Acceleration is deliberately slow and heavy: you climb through the gears with **D** (up) / **A** (down). The engine dies two ways, each after a short warning + hold-ENTER restart:
- **Stall (lug):** revs fall below `LUG_RPM` (0.25 ≈ 650 rpm) — you braked/slowed or sat in too tall a gear without downshifting (1st gear is immune). Realistic: e.g. **cruising 30 km/h in 5th lugs** (you must downshift). The RPM bar drops toward 0 bars and a ~3.5 s `ENGINE STALLING / SHIFT DOWN A` cough warning precedes the stall. RPM is proportional to speed (`rpm = speed / gear.to`) and is shown raw — it CAN read 0.
- **Burn-out (over-rev):** sit on the **redline** under throttle without upshifting → `ENGINE REDLINE / SHIFT UP D`, then it cooks (≈6 s). Only in gears you can upshift out of — the top gear's redline is just the speed limiter, so it's safe (5th's band tops at 130 km/h so the 120 km/h cap sits below redline).

**Synchro shift limits** (per-gear `GEARS[].maxSpeedToShift`, `number | null`): you can only **downshift into** a gear below its limit — **1st < 35 km/h, 2nd < 60, 3rd < 85**; 4th/5th are `null` (no limit). A refused downshift keeps the gear, plays a grind/clunk, and flashes the GEAR readout red (`v.shiftBlocked`). Upshifts are never blocked. Logic is fully config-driven — set all to `null` to drop synchro, all to numbers for a fully synchro'd box. This lets you walk the gears down while braking for ice (low gears unlock as you slow) without ever slamming into an over-revving gear.

RPM is **proportional to road speed** (`rpm = speed / gear.to`, like a real engine) and shown raw, so a too-tall gear reads low (down to 0 bars) and the engine lugs. The model lives in `config.ts` (`GEARS`, `LUG_RPM`, `BOG_RPM`, `BOG_FLOOR`, `RPM_DISPLAY_REDLINE`, `STALL_GRACE_MS`, `REDLINE_RPM`, `REDLINE_BURN_MS`, `OVERREV_ENGINE_BRAKE`, …) and `game/vehicle.ts` (`v.stalled`, `v.stallWarning`, `v.redlineWarning`, `v.stallCause`); RPM also drives engine pitch in `audio/engine.ts`.

**First-person view** — no truck visible; the road scrolls toward you. Manage speed *before* ice, where grip is brutally low (`SURFACE_GRIP.ice` in `config.ts`): steering barely responds and lateral velocity persists → drift. Watch the blinking `ICE AHEAD` strip in the top bar — your cue to slow down (and downshift).

## Phased roadmap

Each phase is self-contained, ends with a runnable build, and leaves the previous scene playable. Time estimates assume part-time work with AI assistance.

> **Recent (post-phase-1, through B47 2026-06-03):** manual 5-speed gearbox, RPM gauge, **A/D** shifting, stall + redline burn-out, synchro downshift limits, tachometer HUD, RPM-driven engine pitch, **hold-ENTER crank start** (1.8 s, both initial and restart), harder torque curve (quadratic bog zone, `BOG_FLOOR=0.12`), surface drag double-penalty fix (mud/sand gears now work correctly), traffic perspective fix (1/z projection matches canisters). Delivery budget 8 min; 20-seed completability sim all pass.

| Phase | Goal | Est. (h) |
|-------|------|----------|
| **0 — Concept & potemkin** ✓ | CLAUDE.md, package skeleton, static frame matching `first_impression.png`. `potemkin.ts` retired in phase 1. | done |
| **1 — Drive scene foundation** ✓ | `DriveScene` with scrolling pseudo-3D road, truck sprite, arrow-key controls (throttle/brake/steer), basic engine drone via Web Audio square-wave osc, ice/asphalt surface generator with grip-affected steering, ICE AHEAD warning blink, tire-screech beeper SFX on ice. SPEED dial wired to real speed; HEADING/FUEL/GRIP cosmetic. Distance + elapsed time tracked. | done |
| **2 — Game over conditions + tune feel** | Off-road detection (lose-control state). Crash/respawn or game-over scene. Tune ACCEL/BRAKE/STEER constants based on real-browser playtest. Bigger truck sprite? Stronger surface visual contrast? | 6–10 |
| **3 — Vehicle physics depth** | Brake distance proportional to speed²; understeer/oversteer split; engine braking; gear-shift feel (audio cue). | 10–14 |
| **4 — More surfaces + visual polish** | Add `snow`, `gravel` to the surface enum. Per-surface dither textures. Wire GRIP gauge to real grip. Surface-transition seam animation. | 8–12 |
| **5 — Fuel + distance + game over** | Fuel decreases scaled by speed. Distance counter tracks real km. Out-of-fuel triggers `GameOverScene`. | 4–6 |
| **6 — Pit-stop scene** | `PitstopScene` pushed between route segments. Menu UI for tyres / fuel / cargo / rest. Decisions tweak next-segment parameters. | 10–14 |
| **7 — Weather + first particles** | `game/weatherFx.ts` game-local: snow particle emitter, wind vector affects steering, visibility curtain limits star draw distance. Decision point: if it works cleanly and is general, propose upstream to zx-kit. | 10–16 |
| **8 — Cargo system** | `game/cargo.ts`: weight affects accel/brake. Random load-shifting event (truck pulls left/right briefly). | 6–10 |
| **9 — Driver fatigue + day/night** | `game/driver.ts`: fatigue curve over real time. Day/night palette swap. Headlights toggle (real-time switch — small in-drive decision). | 10–14 |
| **10 — Risk events** | `game/risk.ts`: tyre blowout, frozen door lock at pit-stop, blizzard window. Procedural triggers based on conditions. | 10–14 |
| **11 — Campaign + save** | Seeded route series. Save profile via zx-kit `save.ts`. Per-route highscores. Unlock progression. | 8–12 |
| **12 — Audio** | AY engine drone modulated by RPM during drive. Beeper SFX library (warning beep, gear shift, ice crack, low-fuel chirp). AY tracker music in menus / pit-stop / intro. | 10–14 |
| **13 — Polish + i18n + release** | `strings.sk.ts`, README, CRT scanline pass, intro story screen, final palette tuning, itch.io/web build. | 8–12 |

**Total estimate: 110–160 hours** = ~3-5 months part-time at 8h/week, or ~4-6 weeks full-time.

**Hard rule: each phase ends runnable.** No phase leaves the game in a half-broken state. If we hit a wall, we revert the phase's branch and reconsider scope.

## Quality bar (1–10 self-assessment as of phase 0)

- **Concept**: 8 — fresh framing, underserved sub-genre.
- **Originality**: 8 — no known ZX ice-road truck game; hybrid pit-stop UX is novel.
- **Period-appropriate graphics**: 9 — potemkin already hits the look; risk is drift during in-motion phases.
- **Replay potential**: 7 — campaign + procedural risks; daily-seed mode could push to 9.
- **Tech difficulty (with AI assist)**: 6 — no 3D, no net, no AI opponents; ~2000-4000 LoC + tests is real software but not over-scoped.
- **Polish risk**: 6 — many small instrument-cluster systems each need buffer.

Re-score after every other phase. Drift triggers a scope cut.

## What NOT to do

- Don't introduce non-Spectrum colours, anti-aliasing, or non-integer scaling.
- Don't add a physics engine. Vehicle physics is ~150 LoC of scalar math, not Box2D.
- Don't add a networking/multiplayer layer.
- Don't bypass zx-kit primitives by drawing directly with `ctx.fillRect` when a kit function exists — if a primitive is missing, propose adding it to zx-kit (with the "second consumer" test above).
- Don't write rotated sprites — ZX-era did not, and we don't either. Use 8-direction sprite sheets.
- Don't render text with anything other than the ZX ROM font (`drawText`, `drawChar`).

## Open decisions (revisit before locking)

- **Day/night cycle?** Affects palette use (dimmed bright colours, bright headlights) — *probably yes*, ties into fatigue.
- **Map / route preview scene?** Nice-to-have, not in first prototype.
- **Highscore upload?** No network. Local only.
- **Slovak first or English first?** English-first per screenshot labels; Slovak as `strings.sk.ts`.

## Reference

- First-impression mock: `first_impression.png` (320×240, captures the target look).
- Concept brief: `first_description.md`.
- Engine: `../zx-kit` — read `../zx-kit/CLAUDE.md` before touching anything kit-shaped.

---
> Source: [zrebec/icehaul](https://github.com/zrebec/icehaul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
