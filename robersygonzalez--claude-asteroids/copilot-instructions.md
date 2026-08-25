## claude-asteroids

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clone of the classic arcade game **Asteroids**, implemented in pure HTML5 Canvas with vanilla JavaScript (ES6+). No frameworks, no bundler, no dependencies, no package.json. The entire game logic lives in a single file, `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There is no build step, no test suite, and no linter configured in this repo.

## Architecture

Everything is in `game.js`, structured as a set of small entity classes plus a global game-state/loop section, in this order:

- **Input** — `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (used for shooting and restart), while `keys[code]` gives held-state (used for rotation/thrust).
- **Utils** — `wrap(v, max)` implements toroidal screen wrapping (used by every moving entity for both x and y); `dist`, `rand`, `randInt`.
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`. All simulation is framerate-independent via `dt` (delta time in seconds).
- **Asteroid sizing** — sizes are `1` (small) to `3` (large), indexing into parallel arrays `RADII`, `SPEEDS`, `POINTS`. `Asteroid.split()` turns one asteroid into two of `size - 1` (size `1` splits into nothing). Asteroid shape is a randomized irregular polygon generated once in the constructor.
- **Game state** — module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) rather than a class/store. `state` is one of `'playing' | 'dead' | 'gameover'` and gates behavior in `update()`.
- **Game flow functions** — `initGame()` (fresh game), `nextLevel()` (called when `asteroids.length === 0`, spawns `3 + level` asteroids), `killShip()` (handles life loss / game over transition), `explode()` (spawns particle burst).
- **update(dt)** — branches on `state` first, then does the per-frame simulation: input → entity updates → bullet/asteroid collision (splits asteroids, awards `POINTS[size]`) → ship/asteroid collision (respects `ship.invincible`) → level-completion check.
- **draw()** — clears canvas, draws particles → asteroids → bullets → ship (back-to-front layering), then `drawHUD()` (score, level, life icons) and any overlay (`drawOverlay` for game over).
- **Main loop** — `requestAnimationFrame(loop)`, computes `dt` clamped to 0.05s max to avoid physics blowups after tab-switch/lag.

Canvas is a fixed 800×600 (`W`, `H` constants); all positions wrap toroidally via `wrap()`, so any new moving entity should use the same pattern.

## Notes

The README (in Spanish) describes power-ups and special asteroid types (e.g. a "shooting star") as features — these are not present in the current `game.js`; treat the README's feature list as aspirational unless you're implementing it.

---
> Source: [robersygonzalez/claude-asteroids](https://github.com/robersygonzalez/claude-asteroids) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
