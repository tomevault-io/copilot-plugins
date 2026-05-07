## sprites-and-rendering

> - `SpriteData = string[][]` - 2D array of hex color strings. Empty string = transparent.

# Sprites & Rendering

## Sprite System
- `SpriteData = string[][]` - 2D array of hex color strings. Empty string = transparent.
- `makeSprite(w, h)` creates the array. `fill(s, x1, y1, x2, y2, color)` and `outline(s, ...)` draw into it.
- `renderSprite()` draws each non-empty pixel as a `scale`-sized rectangle. `imageSmoothingEnabled = false`.
- All sprites are procedural code, no external image files.

## PAL (Palette)
Single `PAL` object in sprites.ts. All colors go here. When adding objects, reuse PAL colors.
Character suit colors: `suitDark`, `suitMid`, `suitLight`, `suitHighlight`, `dressShirt`, `tie`.

## Character Sprites
- `idle` - front-facing standing (skin, suit, tie visible). Uses `makeCharHead` (face with eyes/mouth).
- `back` - rear view (hair, suit back, center seam). Uses raw hair fill, NOT `makeCharHead`.
- `sitType1/sitType2` - seated REAR view with typing animation (arms alternate). Back-of-head like `back`.
- `celebrate` - arms up, stars. Front-facing with `makeCharHead`.
- `walk1/walk2` - walking animation frames. Front-facing with `makeCharHead`.
- `phoning` - standing REAR view, left arm raised holding handset to ear. Back-of-head like `back`.

## Critical: Front vs Back Sprite Construction
- Front-facing sprites call `makeCharHead(s, 0)` → draws face with skin, eyes, mouth.
- Back-facing sprites draw hair/head MANUALLY (fill with `H`/`HH`, ears peeking as `SK`).
- NEVER use `makeCharHead` for back-facing sprites — it draws a visible face.
- When adding a new pose, decide front/back first, then follow the matching pattern.
- `makeSuitTorso` draws the FRONT of the jacket (shirt, tie visible). For back poses, draw jacket back manually (SD fill, SM center seam, SH shoulder highlights).

`facingDir` controls rendering:
- `'back'` = render sprite as-is (back sprites already face away)
- `'left'` = mirror with `ctx.scale(-1, 1)` (idle sprite faces right by default)
- `'right'` = render as-is

## Facing Direction Rules
- Working at desk: always `'back'` (uses sitType sprites)
- Phoning: always `'back'` (uses phoning sprite — character faces desk/phone)
- Idle at coffee/browse/arcade: `'back'` (uses back sprite)
- Idle at water cooler: `'left'`
- Idle at plant: `'right'`
- Walking: follows movement direction (left/right only)
- Default idle: alternates left/right on a timer

## Speech Bubbles
- Clamped to scene bounds: `bx` between `2*scale` and `sceneW - bubbleW - 2*scale`.
- `by` clamped to `>= 2*scale` to prevent top clipping.
- `sceneW = 6 * 32 * scale` (COLS * TILE_SIZE * scale, NOT canvas width).
- Working activities get an icon prefix (emoji). Idle activities don't.
- After initial status text fades (6s), working states auto-cycle "Working." / "Working.." / "Working..." via `speechBubbleTimer = 0.5` loop.

---
> Source: [ofershap/cursor-office](https://github.com/ofershap/cursor-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
