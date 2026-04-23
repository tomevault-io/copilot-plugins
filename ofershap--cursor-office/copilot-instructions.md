## cursor-office

> - 6 columns x 3 rows, each tile 32px. Landscape layout for Cursor's bottom panel.

# Coordinate System & Z-Ordering

## Grid
- 6 columns x 3 rows, each tile 32px. Landscape layout for Cursor's bottom panel.
- Positions use fractional tile coords (e.g., `col: 2.8, row: 1.4`).
- Row 0 = back wall. Higher row = closer to viewer.

## Scaling & Centering
- `scale = Math.min(viewportW / (6*32), viewportH / (3*32))` - fits the smaller axis.
- Scene is centered with `offsetX` and `offsetY`. All mouse events MUST subtract these offsets.
- Canvas size = full viewport. Scene size = `6*32*scale` x `3*32*scale`.

## Z-Ordering (past bug source)
- Objects use UNSCALED `zY` values: `(row + offset) * TILE_SIZE` (no `* scale`).
- Character zY in office.ts: `(position.row + 1) * TILE_SIZE` (also unscaled).
- Both MUST be in the same coordinate space or sorting breaks silently.
- Objects with lower `zY` render first (behind). The `sortable` array in `office.ts` handles this.

## Object Positions
Current layout (col, row) for `createDefaultObjects()`:
- Rug: 1.5, 1.4
- Lamp: 0.5, 0 (ceiling)
- Window: 3.8, 0.15 (back wall, between desk and bookshelf)
- Desk: 1.5, 0.6
- Chair: 1.8, 1.3
- Phone: 1.6, 0.88 (on desk surface, left of keyboard)
- Coffee mug: 3, 0.85
- Bookshelf: 5.1, 0.2
- Water cooler: 4.5, 1.5
- Arcade: 0, 0.8
- Plant: 5.2, 2
- Cat: 3, 1.8

## Placing Objects ON the Desk
The desk sprite (64x44px) starts at (1.5, 0.6). Desk surface is at sprite rows 23-28.
To place an object on the desk surface: `row ≈ 0.6 + (surface_y - obj_height) / 32`.
Phone (16x14px) sits at row 0.88, to the left of the monitor. Mug at row 0.85.
Keep spacing from the monitor stand (centered at sprite x=30-33).

## Waypoint Positions
Character waypoints in `character.ts` must align with object positions.
"In front of" means HIGHER row (closer to viewer), facing 'back'.
`OBJECT_POSITIONS` map and `IDLE_WAYPOINTS` array must stay in sync with `createDefaultObjects()`.

## Key Character Positions
- `DESK_COL = 1.9, DESK_ROW = 1.05` — character walks here when working. Chair is at (1.8, 1.3).
- `PHONE_COL = 1.6, PHONE_ROW = 1.05` — character walks here when phoning (near left side of desk).
- `IDLE_COL = 3, IDLE_ROW = 1.8` — center of room, default idle position.
- `CELEBRATE_COL = 3, CELEBRATE_ROW = 1.8` — same as idle center.
- `isAtDesk()` and `isAtPhone()` both use 0.5 tile tolerance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ofershap) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
