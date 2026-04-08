## playbook

> Interactive football playbook drawing tool. Coaches draw plays on a half-field canvas: place players, draw routes, add sticky notes, export to PDF. Supports up to 3 plays, built for 7v7 flag football.

# Playbook Proto — Claude Guide

## What this app is
Interactive football playbook drawing tool. Coaches draw plays on a half-field canvas: place players, draw routes, add sticky notes, export to PDF. Supports up to 3 plays, built for 7v7 flag football.

## Stack
- React 18 + TypeScript (strict), Vite, jsPDF + html2canvas
- No external state library — all state in `App.tsx` via `useState`
- Base path `/Playbook/` (GitHub Pages deployment)

## Dev commands
```bash
npm run dev      # localhost:5173
npm run build    # tsc + vite build → /dist
npm run deploy   # build + gh-pages -d dist
```

## Directory map

| Path | What lives here |
|------|----------------|
| `src/App.tsx` | **State hub.** All play/stroke/player state, undo stack, keyboard shortcuts, all update handlers |
| `src/types.ts` | All TypeScript types (`Play`, `Stroke`, `Player`, `StickyNote`, etc.) |
| `src/constants.ts` | Route colors, rosters, `generateId`, `MAX_PLAYS = 3` |
| `src/storage.ts` | localStorage read/write for plays |
| `src/components/FieldCanvas.tsx` | **Core canvas.** All mouse/touch handlers, drawing in-progress state |
| `src/components/ToolRail.tsx` | Left toolbar — 6 tools with flyout panels |
| `src/components/TopBar.tsx` | Play tabs (create/rename/delete), first-down yardage slider |
| `src/components/Sidebar.tsx` | Formation, Situation fields, Export PDF button |
| `src/components/CanvasArea.tsx` | Container wrapping FieldCanvas + sticky notes |
| `src/components/PlayerContextMenu.tsx` | Popup for changing player shape/color/label |
| `src/components/EmptyState.tsx` | Shown when no plays exist |
| `src/canvas/drawField.ts` | Field background, yard lines, end zone, LOS, first-down marker |
| `src/canvas/drawPlayers.ts` | Player shapes (circle, square, triangle, X, star) + contrast-aware labels |
| `src/canvas/drawStrokes.ts` | Routes — solid/dashed/arrow, branch routes, yardage annotations |
| `src/canvas/drawStickyNotes.ts` | Sticky note render + hit-testing |
| `src/utils/pathUtils.ts` | `getAdjustedStrokePoints()` — anchored route offset math |
| `src/utils/colorUtils.ts` | Luminance + contrast helpers |
| `src/utils/fieldGeometry.ts` | Field dimensions and coordinate helpers |
| `src/utils/exportPDF.ts` | PDF layout using jsPDF + html2canvas |

## Key data structures (from `src/types.ts`)

```typescript
Play       { id, name, createdAt, strokes, players, stickyNotes, notes, formation?, situation? }
Stroke     { id, points[], color, width, tool, lineStyle, anchoredPlayerId?, parentStrokeId?, branchPointIndex?, annotations? }
Player     { id, x, y, team, label, number, shape, color? }
StickyNote { id, x, y, text, color }
```

## Data flow
```
Mouse/touch event (FieldCanvas)
  → callback prop → App.tsx handler
    → updateActivePlay() mutates plays[]
      → savePlays() persists to localStorage
        → setPlays() triggers re-render
          → FieldCanvas useEffect redraws canvas
```

## Important patterns & gotchas

**Anchored routes** — Strokes store `anchoredPlayerId`. When a player moves, `getAdjustedStrokePoints()` in `pathUtils.ts` recalculates all its routes. Deleting a player does NOT delete its anchored strokes — they just stop moving.

**Branch routes** — A stroke can have `parentStrokeId` + `branchPointIndex` to branch off another route at a specific point.

**Canvas always full-redraw** — No layer caching. Every change redraws everything. Keep draw functions fast.

**Coordinate system** — Field is 30 yards wide × 40 yards tall (30 playing + 10 end zone). Mouse coords are converted to canvas space via CSS scaling.

**Undo stack** — In `App.tsx`, max 20 snapshots per active play. No redo.

**Keyboard shortcuts** — `D`=draw, `P`=player, `N`=note, `Z`=zone, `E`=erase, `S`=select, `Cmd+Z`=undo, `Delete`=remove selected.

**CSS theme** — Dark-only. Variables in `src/index.css`: `--bg`, `--text`, `--accent` (#e8ff47 lime). No light mode.

**localStorage** — Saved on every play change. No quota-exceeded handling. MAX_PLAYS=3 keeps it manageable.

**Route annotations** — Double-click on a route to add a yardage label. Stored as normalized t-value (0–1) along the path.

**Export PDF** — Only exports the active play. Image quality is fixed at 1.0 DPI ratio.

**Player shapes** — `circle`, `square`, `triangle`, `x`, `star`. Default offense=`#caff6f` (lime), defense=`#ff4757` (red).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NedyUdombat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NedyUdombat)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
