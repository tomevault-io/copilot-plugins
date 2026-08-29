## shotgun-ninjas-virtual-studio

> This repository contains Shotgun Ninjas Virtual Studio, a free browser-based DAW/web audio app.

# AGENTS.md - Shotgun Ninjas Virtual Studio Stabilization Rules

## Mission

This repository contains Shotgun Ninjas Virtual Studio, a free browser-based DAW/web audio app.

The current priority is stability, latency reduction, crash prevention, and performance repair.

Do not add creative features during stabilization work.

## Business Rule

Shotgun Ninjas Virtual Studio must remain free.

Forbidden:
- Stripe
- paid tiers
- subscriptions
- locked exports
- forced accounts
- ads
- upgrade prompts
- paywall language
- artificial usage limits

Allowed:
- stability fixes
- performance mode
- diagnostics
- bug repair
- documentation
- accessibility improvements
- user-facing error handling
- free product policy preservation

## Stabilization Priorities

Fix in this order:

1. Page unresponsive crashes
2. Main-thread blocking
3. Audio latency/stutter
4. React render storms
5. Tone/Web Audio lifecycle leaks
6. Duplicate scheduler/transport events
7. Autosave/project storage freezes
8. WAV export memory explosions
9. Sample import/waveform freezes
10. Visualizer/meter/particle performance
11. PWA/service worker stale cache bugs
12. CSS layout/paint cost

## Non-Negotiable Rules

- Do not rewrite the app from scratch.
- Do not migrate frameworks.
- Do not redesign the UI.
- Do not add new creative features.
- Do not disable core features unless absolutely necessary and documented.
- Do not claim a fix unless it is tested.
- Do not hide failures behind spinners.
- Prefer small, targeted patches.
- Keep audio timing separate from React render state.
- Do not create Tone/Web Audio nodes inside React render bodies.
- Dispose or disconnect audio nodes when tracks/kits/effects are replaced or deleted.
- Store and clear Tone.Transport schedule IDs.
- Prevent duplicate scheduler loops.
- Avoid high-frequency React state updates.
- Use refs/canvas/shared tickers for meters and playheads.
- Lazy-load heavy panels.
- Debounce autosave and avoid rewriting unchanged blobs.
- Keep user projects safe.

## Required Documentation

Every stabilization PR/change must update or create:

- `PERF_BASELINE.md`
- `PERFORMANCE_AUDIT.md`
- `PERFORMANCE_FIXES.md`

## Required Testing

At minimum run:

```bash
npm install
npm run build
npm run lint || true
npm run test || true
npm run preview || true

If scripts differ, inspect package.json and use the available equivalents.

Required Manual Test Checklist

Document results for:

App loads without console errors.
Enable Audio works.
Spacebar play/pause works.
Stop releases audio.
Panic stops all audio.
Demo project loads without freezing.
Playback runs 10 minutes without page unresponsive dialog.
Mixer opens during playback without freeze.
Visualizer opens during playback without freeze.
Normal sample import does not freeze page.
Project save does not freeze page.
Project load does not freeze page.
JSON export works.
WAV export works or limitation is clearly documented.
Autosave skips unchanged projects.
Hidden visual panels stop animation loops.
Repeated kit/instrument switching does not steadily leak memory.
Repeated project load/unload does not stack Tone.Transport events.
Performance Mode reduces visual load.
Production build behaves better or equal to dev build.
Final Response Format

When finished, respond with:

Root causes confirmed
Fixes applied
Files changed
Tests run
Before/after observations
Known remaining risks
Recommended next action

---
> Source: [shotgunsensei/Shotgun-Ninjas-Virtual-Studio](https://github.com/shotgunsensei/Shotgun-Ninjas-Virtual-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
