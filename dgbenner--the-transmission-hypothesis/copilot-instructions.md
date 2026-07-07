## the-transmission-hypothesis

> A continuous web-based video feed styled after late-night cable and pirate

# Transmission Hypothesis — Project Brief for Claude Code

## What this is
A continuous web-based video feed styled after late-night cable and pirate
transmissions. Live at thetransmissionhypothesis.com. Hosted on GitHub Pages,
videos on Cloudflare R2, custom domain via Cloudflare Registrar. The site
exists to feel like something you stumbled onto — not something that was
designed for you.

## Aesthetic guardrails
The site lives or dies on tone. A "good" haunted transmission effect feels
like signal degradation, broadcast intrusion, or something the transmission
itself is doing to you — not like a web design flourish. Reference points:
analog tape damage, VHS tracking errors, pirate broadcast hijacks, the
texture of a TV picking up a station it shouldn't.

Effects should NOT feel like:
- Modern web animation (smooth easings, polished transitions)
- Generic "glitch" filters (RGB split as a one-shot, chromatic aberration
  as decoration)
- Anything that announces itself as a feature

Effects SHOULD feel like:
- Something wrong with the transmission, briefly, and then gone
- Texture that rewards attention from repeat viewers without demanding it
- Variations on existing effects in the catalog rather than novel categories

When in doubt: if a casual visitor would think "neat effect," it's wrong.
If a repeat viewer would think "wait, did that just—", it's right.

## Effects system

All effects live as plain top-level functions inside `index.html`'s main
inline `<script>` block. There is no registry, no manifest, no plugin
architecture — every effect is wired up by hand. Adding a new effect
means adding (1) a trigger function, (2) a scheduler, and (3) a line
inside `startHaunting()` to kick off the first run.

### The pattern

For every effect in the catalog there are usually three functions:

- **`triggerXxxEvent()`** — fires the effect once. Reads the suppression
  guard (`window.shouldSuppressEffects()`) and bails immediately if it
  returns true. Otherwise builds an effect "phrase" or config, applies
  visual changes, and schedules its own cleanup via `setTimeout` /
  CSS class removal / animation end.
- **`scheduleXxxEvent()`** — wraps `setTimeout` around `triggerXxxEvent`,
  picks a random delay inside the effect's interval window, and recurses
  by calling itself at the end. Self-perpetuating loop.
- **`triggerXxx(...)` / `executeXxxPhrase(...)`** (sometimes) — internal
  helpers used by the trigger to assemble the phrase or render a single
  beat of the effect. Not all effects have these; simple ones inline
  everything in the trigger.

### Boot sequence

`setTimeout(startHaunting, 2000)` runs once on load. `startHaunting()`:

1. Calls `clearEffect()` to reset any leftover scanline state.
2. Calls `scheduleEvent('minimal'|'moderate'|'major'|'critical'|'catastrophic')`
   for each of the five intensity tiers of the master haunted-event
   system. Those five are configured by the `INTENSITY_LEVELS` object
   which sets per-tier interval, variance, duration, and intensity ranges.
3. Schedules every other effect with a one-shot `setTimeout` using a
   randomized initial delay (the "staggered start"), so nothing fires at
   the same moment. Each of those callbacks then either calls its
   `scheduleXxxEvent` to start the recurring loop, or kicks off the loop
   internally inside the trigger.

Effects currently wired in `startHaunting()`: ghost, static phrase, color
streak, lens flare, critical flare burst, scramble, recession, pixelation
/ random pixel effect, webcam interrupt, glass sphere, color burst, grid
displacement, page drift, filter wash, film gate. Plus the master
`triggerEvent(level)` haunted scanline system which is scheduled per
tier. Flash frames are bootstrapped separately at the end of their own
IIFE (`scheduleNext()` recursing on its own timer), and the title
glitch system likewise lives in its own block at the bottom of the file.

### Suppression

`window.shouldSuppressEffects()` returns `true` when **either**
`isEventActive` (a designated transmission is playing) or
`isMidLengthActive` (a mid-length unscheduled broadcast is playing) is
true. Every effect's `triggerXxxEvent` is expected to check this guard
on its first line and return early if true. The schedulers continue to
run regardless, so once long-form playback ends the effects resume on
their normal cadence. There are also per-effect mutual-exclusion flags
(`webcamActive`, `pixelActive`, `corruptionActive`) that prevent
overlapping heavy effects from stomping each other.

### Cleanup / unloading

Effects clean themselves up; there is no shared destructor. The two
patterns in use:

- **Time-bounded effects** (the haunted scanline system, streaks,
  flares, color burst, etc.) call `setTimeout(clearXxx, duration)` at
  trigger time and let the timer take care of removal. `clearEffect()`
  is the master reset for the scanline overlay; other effects either
  remove the DOM nodes they injected or strip a class they added.
- **State-flag effects** (webcam interrupt, pixelation, signal
  corruption) flip a module-level `xxxActive = true` flag, run their
  own internal sequence which checks that flag at every step (so it can
  be cancelled mid-flight by setting the flag false), and then flip the
  flag back to false at the end. Cancellation paths exist on
  `loadTransmission`, scheduled-event entry, and signal-lost recovery.

### Manual triggers

There's a global `keydown` listener (capture phase, requires Shift) that
maps single letters to manual fires of most effects for testing:
B (cycle haunt levels), G (ghost), S (static), X (clear static),
C (streak), F (lens flare phrase), D (scramble), E (recession),
P (pixel/corruption random), Z (webcam), O (glass sphere), R (color
burst), T (grid displacement), W (page drift), and so on. Disabled
when `window._isOBS` is set.

### Where to add a new effect

1. Add the trigger function near the existing effect family it most
   resembles.
2. Add a scheduler that loops itself.
3. Add a staggered initial `setTimeout(triggerXxxEvent, randomDelay)`
   inside `startHaunting()`.
4. Make sure the trigger's first line is the suppression guard:
   `if (window.shouldSuppressEffects && window.shouldSuppressEffects()) return;`
5. If the effect can stomp others, gate it with a mutual-exclusion flag.
6. If you want a debug shortcut, add a Shift-letter case to the keydown
   listener.

## Flash Frames spec
~50-80ms full-screen image flashes. Frequency: every 5-7 minutes during
normal operation. Manual trigger: Shift+F. Disabled during designated
transmissions (those play uninterrupted).

## Designated transmissions
Scheduled at GMT intervals. The scheduling logic is load-bearing — do not
modify it without an explicit ask. Effects must not fire during a
designated transmission window.

## Branching rule
For interactive sessions, commit directly to main. Dan will explicitly
ask for a branch when one is wanted (e.g. for risky changes, or for
parallel exploration of an idea).

For unattended/batch runs (overnight effects batches, multi-agent runs,
anything where Dan is not in the loop on each commit) — branch as
`agent/<short-description>` or `overnight/<batch-name>`, push the branch,
do not merge. Dan reviews and merges through GitHub.

## Verification
Before declaring an effect done, run the verification script at
scripts/verify-effects.js. It loads each effect in a headless browser,
captures clips to /review/, and checks for console errors and cleanup
failures. An effect that fails verification is not done — debug up to
three attempts, then mark it as abandoned in the summary and move on.

## Output expectations for batch runs
At the end of a batch, write summary.md in the branch root. For each
effect: name, what you were going for, whether it passed verification,
anything you want Dan to look at specifically. Keep it brief — Dan is
reviewing the clips, not the prose.

## Out of scope without explicit ask
- Analytics changes (StatCounter, Microsoft Clarity)
- R2 video uploads or deletions
- DNS or Cloudflare config
- The designated transmission schedule
- Anything outside this repo

---
> Source: [dgbenner/the-transmission-hypothesis](https://github.com/dgbenner/the-transmission-hypothesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
