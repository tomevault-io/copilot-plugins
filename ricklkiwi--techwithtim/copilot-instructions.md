## techwithtim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

Open `index.html` directly in a browser — no build step, no server required:

```bash
open index.html
```

## Repository Structure

This is a two-file vanilla JS browser game — no framework, no bundler, no dependencies.

- `index.html` — Canvas shell (800×600), CSS (`cursor: none`, dark bg), loads `game.js`
- `game.js` — All game logic: constants, state machine, all classes and modules

## Architecture (`game.js`)

### State Machine
`GameState.state` drives the entire loop: `MENU → PLAYING → LEVEL_TRANSITION → PLAYING → … → GAME_OVER`

Transitions:
- `GameState.startGame()` — resets and enters PLAYING
- `GameState.nextLevel()` — advances level index; enters LEVEL_TRANSITION or GAME_OVER (victory)
- `GameState.finishTransition()` — enters PLAYING and calls `initLevel()`
- `GameState.reset()` — returns to MENU

### Game Loop (`loop()`)
Single `requestAnimationFrame` loop. Delta time is computed and capped at 100ms, then normalized to 60fps units (`dt = rawDt / (1000/60)`). All movement/timer logic multiplies by `dt`.

Order each frame: update input → update state-specific logic → render.

### Key Globals
- `bullets` — array of `Bullet` instances, reset on `initLevel()`
- `WaveManager` — singleton managing enemy spawning and wave progression
- `Particles` — singleton flat-array particle system
- `Player` — singleton player object
- `Input` — singleton tracking keyboard and mouse state; call `Input.clearFrame()` each tick to reset one-shot events

### Level Data
`LEVELS[]` is a plain array of config objects. Each entry has `enemySpeedMult`, `enemyHpMult`, and `waves[]`. Each wave has an `enemies[]` array of `{ type, count }` and a `spawnInterval` (frames between spawns). `GameState.levelIndex` indexes into this array.

### Enemy Types
Defined via a `type` string passed to the `Enemy` constructor: `"basic"`, `"fast"`, `"tank"`. Stats (speed, hp, radius, score) are set in the constructor switch. Draw methods are `_drawBasic`, `_drawFast`, `_drawTank` — all procedural canvas shapes, no image assets.

### Collision Detection
`circleCircle(a, b)` — `Math.hypot` distance check against `a.radius + b.radius`. Called in `checkCollisions()` for bullets↔enemies and enemies↔player.

## GitHub Integration

A Claude Code GitHub Actions workflow (`.github/workflows/claude.yml`) is configured. Mention `@claude` in any issue, PR, or comment to trigger it. Requires `ANTHROPIC_API_KEY` set as a repository secret.

## Git Workflow — Always Commit and Push

**Every meaningful change must be committed and pushed to GitHub immediately.** This ensures no work is ever lost and the repo always reflects the current state of the project.

### When to commit
- After completing any feature, fix, or improvement — no matter how small
- After updating any config or documentation file
- Never leave modified files sitting uncommitted at the end of a session

### Commit message format
```
Short imperative summary of what changed and why (≤72 chars)

- Bullet describing the first specific change
- Bullet describing the second specific change
- Group related changes together

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Rules:
- First line answers "what does this commit do?" — not "what files changed?"
- Body bullets explain the *why* and *what*, not just file names
- Always include the `Co-Authored-By` trailer
- Stage specific files by name — never `git add .` or `git add -A`
- Always push immediately after committing: `git push origin master`

### Workflow per change
```bash
git add <specific files>
git commit -m "$(cat <<'EOF'
Summary line

- Change detail 1
- Change detail 2

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
git push origin master
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ricklkiwi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
