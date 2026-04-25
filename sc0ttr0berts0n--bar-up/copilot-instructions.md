## bar-up

> - **Stack**: Vue 3 + TypeScript + Vite + Pixi.js v8 + PeerJS (WebRTC P2P multiplayer)

# Drink Up! — Agent Instructions

## Quick Start

- **Stack**: Vue 3 + TypeScript + Vite + Pixi.js v8 + PeerJS (WebRTC P2P multiplayer)
- **Build check**: `npx vite build` (the `vue-tsc` in `npm run build` has a known `.vue` module error — use `npx vite build` for validation)
- **Dev server**: `npm run dev` (HTTPS via vite-plugin-mkcert)
- **No automated tests** — manual browser testing required (WebRTC + Pixi canvas)

## Architecture

Authoritative host simulation. Host runs game logic at 60Hz tick, broadcasts state at 20Hz to all clients via PeerJS.

```
src/
├── game/
│   ├── Network/
│   │   ├── Communicator/     # PeerJS networking, packet types, handlers
│   │   └── Server/           # Host-only game logic
│   │       ├── Engine.ts     # Main game engine (tick, interactions, state)
│   │       ├── GameObjects/  # Bartender, Guest, Appliance, Item, DrinkCrafter, GuestSpawner, ShiftManager
│   │       └── Game.ts       # Server game loop + state broadcast
│   ├── Shared/               # Types/config shared by server + client
│   │   ├── ApplianceTypes.ts # Appliance enum, configs, seat offsets
│   │   ├── BarLayout.ts      # 20x15 grid layout, appliance placements
│   │   ├── DrinkRecipes.ts   # 13 recipes, variant configs, grab configs
│   │   ├── GameSettings.ts   # All tunable constants
│   │   ├── GuestTypes.ts     # Guest status enum, state interface
│   │   ├── ItemTypes.ts      # Item enum, GLASS_TYPES set, ITEM_DISPLAY colors
│   │   └── TileTypes.ts      # Tile zones, directions
│   ├── View/                 # Pixi.js client rendering
│   │   ├── Game.ts           # Client input handling, state sync, sub-menu logic
│   │   ├── Level.ts          # Tile grid, appliances, messes, items on surfaces
│   │   ├── GuestView.ts      # Guest rendering (body, status, order bubble, drink glass, bars)
│   │   ├── Player/BartenderView.ts
│   │   └── ItemView.ts
│   └── Utils/                # Singleton, World, Random, Lerp
├── components/UI/            # Vue HUD overlays
│   ├── HUD.vue               # Held item, money, phase, timer
│   ├── SubMenu.vue           # Variant selection (positioned above player)
│   └── RecipeGuide.vue       # Recipe reference panel
└── store/global.ts           # Reactive Vue store (HUD data, submenu state)
```

## Key Patterns

### Two-Button Interaction System
- **E key** = Grab/Drop (pick up items, put down on surfaces, sub-menu for glass shelf)
- **Space** = Action/Interact (take orders, serve drinks, craft at appliances, sub-menu for variant selection)
- Sub-menus: Arrow keys / WASD navigate, 1-4 direct select, Enter/Space/E confirm, Escape cancel

### Context-Sensitive Sub-Menu Flow
1. Client detects situation (holding glass + facing appliance, or empty-handed + glass shelf)
2. Vue SubMenu overlay opens at player's screen position
3. Player selects variant
4. `CLIENT_SELECT` packet sent to server
5. `Engine.bartenderSelect()` handles both craft mode (transform item) and grab mode (spawn item)

### Item Lifecycle
Glass Shelf → Pick glass type (E sub-menu) → Appliance (Space sub-menu) → Transform to drink → Serve to guest → Drink sits on counter during DRINKING → Transforms to DIRTY_GLASS when finished → Pick up (E) → Sink (Space) → Consumed

### Guest Lifecycle
Spawn at entrance → Walk to random seat → DECIDING → READY_TO_ORDER → WAITING_FOR_ORDER → DRINKING → (reorder or leave) → LEAVING → Removed

### State Data Sync
When adding fields to any state interface (Guest, Player, Item, Appliance), update THREE places:
1. The interface in `Shared/` (e.g., `IGuestStateData`)
2. The `state` getter in the server class (e.g., `Guest.ts`)
3. The Zod schema in `ServerPacketHandler.ts` (uses `z.nativeEnum` for enums — auto-updates)

### Rendering Items on Surfaces
Items placed on appliances (counter, service bar) render as color-coded pills in `Level.ts` update loop. The item pool grows as needed.

## Layout

20x15 grid. Row 0: wall. Rows 1-2: employee area (appliances). Row 3: bar counter (14 seats, x=2 to x=15). Row 4: walking lane. Rows 5-12: seating (hightops at y=7, tables at y=10). Row 13: walking lane. Row 14: wall + entrance at center.

Service bar at (16,2) is 2-wide — counters at x=16,17 are removed to keep that area clear.

## Roadmap

See `roadmap.md` for the full 6-phase feature roadmap. Phase 1 + Phase 2 complete. Phase 3 (economy & tipping) is next.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sc0ttr0berts0n) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
