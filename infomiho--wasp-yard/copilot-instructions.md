## wasp-yard

> Architecture and mechanics are in `README.md`. This file covers what is not visible in the code.

# AGENTS.md

Architecture and mechanics are in `README.md`. This file covers what is not visible in the code.

## Commands

```
npm run dev     vite, port 5173
npm test        vitest, 47 tests
npm run build
```

`test/playthrough.test.jsx` drives the whole game through the real DOM in jsdom. It uses real
timers, so it takes about 20 seconds. Any change to card timing has to go through the
`FAILURE_CARD_DELAY` constant exported from `src/engine/engine.js`, which the test imports
rather than hard-coding a number.

## Accuracy

Every mechanic and every line of copy describes something real in the Wasp repo: a shipped
behaviour, a merged PR, or the proposed design in `waspc/wireframes/sdk-runtime-boundary-evolution.html`.
Nothing is invented to make a point. Levels carry an `act` label stating their status
(`SHIPPED TODAY`, `TRIED, NEITHER SHIPPED`, `PROPOSED, NOT BUILT`); a claim that cannot be
placed under one of those labels does not belong in the game.

Verify against the `wasp` checkout before adding a mechanic. `initializeServerRuntime`, for
one, exists only in the wireframe, not in shipped code.

## Prose

Plain, calm, technical, impersonal. No punchy writing: no sentence fragments used for
rhythm, no m-dashes, no exclamation marks, no invented metaphors, no "N things, in order"
framings. Instructions name the exact control to click. Dialogs with more than three facts
use bullet lists.

All user-facing strings live in `src/engine/copy.js`, `src/engine/walls.js`, and the
`narrative` block of each level in `src/engine/levels.js`.

## Layout changes

The board is generated from per-level grid coordinates in `levels.js`. Cable crossings are
the usual regression. Screenshots are the working feedback loop: drive the level with
Playwright, wire the cables, and screenshot `.board-content`. Level state can be set
directly by writing `waspyard.save.v1` to localStorage and reloading.

Two stacking rules that are easy to break: the cable SVG sits above the lanes, so anything
that must read over a cable needs its own lane-level z-index (see `.lane:has(.drop)`); and
dialog cards dock to the board's top edge via `--dock-top`, measured on mount, so they never
cover the header or the console.

---
> Source: [infomiho/wasp-yard](https://github.com/infomiho/wasp-yard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
