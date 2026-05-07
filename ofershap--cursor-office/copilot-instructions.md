## interactive-objects

> Each object is created by a `create*()` function in `objects.ts` returning `InteractiveObject`.

# Interactive Objects

## Object Factory Pattern
Each object is created by a `create*()` function in `objects.ts` returning `InteractiveObject`.
The `render` function receives `(ctx, obj, tick, scale)` — it draws relative to `obj.position` using `scale`.
`onClick` mutates `obj.state` (toggle, cycle, etc.) and can mutate `OfficeState`.

## Registering External Objects
`window.agentArcade.registerObject(obj)` — lets external code inject custom objects.
`registerBackground(bg)` — replaces floor/wall rendering with custom `BackgroundRenderer`.
This is the extensibility API for future plugins/themes.

## Current Objects & Their States

| Object | `state` key | Behavior |
|--------|------------|----------|
| lamp | `on: boolean` | Toggle. Sets `office.dimmed`. Flash effect on toggle. |
| window | `open: boolean` | Toggle curtains. Sky renders day/night/sunset based on system clock. Light beam when open + daytime. |
| arcade | `game: number` | Cycles through 3 mini-game animations (Space Invaders, Tetris, Pong). Scanline + glow effects. |
| coffee | `steamTimer: number` | Click → steam particles rise for 3 seconds. |
| cat | `targetCol/Row, moveTimer, nudged, purring, facingRight` | Autonomous wandering AI. Click → nudge bounce + purr hearts. |
| plant | `stage, clicks, bounce` | Click → water. After 4 clicks → stage 1, 10 clicks → stage 2 (flowers + sparkles). |
| watercooler | `bubbleTimer: number` | Click → rising bubbles for 2 seconds. |
| phone | `ringing: boolean, ringTimer: number` | Vibrates when `ringing=true`. Synced with character phoning activity via message handler in `index.ts`. |
| bookshelf | `selected, bubbleTimer, title` | Click → cycles through book titles, shows in custom wide bubble. |
| rug | (none) | Non-interactive. Woven texture with diamond pattern and tassels. |

## Hitbox Gotchas
- Hitbox `{ w, h }` is in PIXEL space (unscaled). Hit test multiplies by `scale`.
- Position is in TILE coords, so pixel position = `col * TILE_SIZE * scale`.
- Small objects (cat, plant) need oversized hitboxes for usability. Aim for at least 20x14 for small items.
- Hitbox anchors at top-left of the object's tile position.

## Adding a New Object
1. Create sprite(s) in `sprites.ts` using `makeSprite` / `fill` / `outline`.
2. Add factory function in `objects.ts` with position, hitbox, zY, state, onClick, render.
3. Add to `createDefaultObjects()` array.
4. Add entry in `OBJECT_POSITIONS` (character.ts) if the character should walk to it on click.
5. Add corresponding waypoint in `IDLE_WAYPOINTS` if the character should visit it during idle patrol.
6. Set `zY = (row + some_offset) * TILE_SIZE` (unscaled) for correct depth sorting.

---
> Source: [ofershap/cursor-office](https://github.com/ofershap/cursor-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
