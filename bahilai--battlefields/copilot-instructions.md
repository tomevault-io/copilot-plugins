## battlefields

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BattleFields is a hex-based tactical game built with Phaser 3. It features turn-based combat between green and red armies on a hex grid battlefield with terrain, static weapons, dice mechanics, and a general/company system.

## Running the Game

This is a client-side only Phaser game. To run:
1. Serve the project with any local web server from the root directory
2. Open `index.html` in a browser
3. The game uses ES6 modules, so a web server is required (not `file://`)

No build process, package.json, or npm commands exist - this is vanilla JavaScript.

## Architecture

### Core Systems (src/core/)

**HexGrid** - Converts between world coordinates and hex grid (q,r) coordinates. Uses offset coordinates with odd-row stagger. Key methods:
- `toWorld(q, r)` - hex coords to pixel position
- `fromWorld(x, y)` - pixel position to hex coords
- `neighbors(q, r)` - returns array of valid neighbor hex coordinates
- `hexDistance(q1, r1, q2, r2)` - cube distance between hexes

**TerrainMap** - Extracts terrain types from the Tiled hex layer. Terrain types:
- `void` - impassable
- `water` - impassable (GIDs: 1,7,8,9,12,14,15)
- `forest` - movement cost 2 (GID: 11)
- `trench` - movement cost 1, tanks cannot stop here (GIDs: 3221225489-91, 17-19)
- `grass` - movement cost 1 (GID: 16)

Static weapons on separate layers: `machine-gun` and `antitank-gun`

**Pathfinder** - BFS-based movement calculation. Takes units array as `getOccupiedSet` callback. Special rules:
- Tanks crossing trenches need >1 movement remaining and cannot end on trenches
- Considers terrain costs and occupied hexes
- Returns Map of reachable hexes with `costLeft` for each

**CombatSystem** - Handles attacks and damage. Damage model:
- First hit: `ready` → `wounded` (plays damaged animations, no tint)
- Second hit on infantry: `wounded` → `dead` (plays death animation)
- Second hit on tank: `wounded` → `dead` (shows static dead texture)
- Dead units have `attacks=0`, `movesLeft=0`, sprite non-interactive

**Unit** - Base unit class with:
- `side`: 'green' or 'red'
- `kind`: 'infantry' or 'tank'
- `state`: 'ready', 'wounded', or 'dead'
- Movement with tweened animation based on direction
- Directional animations: infantry has left/right, tanks have left/right/up/down
- Separate animation sets for normal and wounded states

**UnitFactory** - Creates Unit instances with scene and grid context

**Hud** - Bottom UI bar (280px high) showing:
- General portrait (156x156px, left side)
- Two company icons (artillery, supply) - 156x156px each
- Info panel (turn, unit stats, event log)
- Help button (yellow '?' under info panel)

**Dice** - Physics-based dice rolling system:
- Draggable dice with swipe-to-throw mechanic
- Real-time physics simulation: gravity, velocity, friction, bounce
- Drag strength determines throw velocity (max 2000 units/sec)
- Collision detection with screen boundaries
- Random value changes during roll, settles on final value
- Auto-returns to home position after 1 second
- Roll determines active company (1-3: artillery, 4-6: supply)
- Each round: roll selects general/colonel (values 1-6 determine different colonel types)
- Physics updates in `_updatePhysics(time, delta)` via scene events
- Callbacks: `onDiceResolved(value)`, `onCompanyUsed(kind)`

### Scene Structure (src/scenes/)

**BattleScene** - Main game scene. Initialization order critical:
1. Load assets (background, tilemap JSON, tilesets, sprite frames)
2. Create tilemap with layers: hexes, bases, flags, machine-gun, antitank-gun
3. Calculate dynamic scale to fit screen (720x1280 portrait, 280px HUD reserved)
4. Apply tileoffset compensation for red bases layer (+16x, -7y scaled)
5. Initialize HexGrid, UnitFactory, TerrainMap
6. Create Hud and Dice (before pathfinder/combat)
7. Wire callbacks between Dice ↔ Hud
8. Initialize Pathfinder and CombatSystem with units array
9. Spawn initial units
10. Register input handlers

Turn system:

**Round phases (each round):**
1. **Initiative** (only Round 1) - Dice roll determines who goes first for entire game
2. **Commander Hiring** (every round) - Both players roll to get colonel with bonuses
3. **Company Activation** (skipped in first day of Round 1) - Roll for artillery or supply
4. **Unit Actions** - Players alternate 5 actions each (one unit per action)
5. **Night Phase** - Roll determines Rest (company activation) or Night Battle (repeat day)

- **Initiative**: Determined only once in Round 1. Winner gets first turn for the entire game.
- **Commander Hiring**: Happens EVERY round. Each player rolls for new colonel (1-6 = different bonuses)
- **Manual turn switching**: Players click End Turn button ("▶") to finish their turn (no auto-switching)
- `currentPlayer`: 'green' or 'red'
- Units of current player can be selected and moved/attacked
- **Unit actions per turn**:
  - Can move ONCE (remaining movement points are lost after first move, tracked by `hasMoved` flag)
  - Can attack multiple times (based on unit type: infantry=2, tank=1, machinegun=4)
  - Movement and attacks can be combined in same turn
  - If unit used static weapon (MG) and left it, cannot attack this turn (`usedStaticWeapon` flag)
- End Turn button positioned above HUD on right side, enabled only during actions phase
- Switching player resets all that player's units' `attacks`, `movesLeft`, `hasMoved`, `usedStaticWeapon`
- `initiativeDetermined`: Boolean flag tracking if initiative was rolled (prevents re-rolling)
- `initiativeWinner`: Player who won initiative in Round 1, preserved throughout the game
- `playerTurnFinished`: Tracks which players have ended their turn in actions phase

**AnimationDemo** - Separate scene for testing animations (not actively used)

### Key Game Flow

1. Player taps unit → `_selectUnit()` if it's their side
2. Pathfinder computes reachable hexes and highlights them (black overlay) - only if `!unit.hasMoved`
3. Enemies in attack range highlighted red
4. Usable MG/AT positions highlighted blue - only if `!unit.hasMoved`
5. Player taps destination:
   - Empty reachable hex → move unit there (only if `!unit.hasMoved`), sets `unit.hasMoved = true`, `unit.movesLeft = 0`
   - Enemy in range → `combat.performAttack()` (checks `unit.usedStaticWeapon`)
6. After action, `_checkTurnEnd()` calls `turnSystem.recordAction()` (no longer auto-switches)
7. Player clicks End Turn button → `turnSystem.endPlayerTurn()` → switches to other player or ends phase

### Animation System

All animations created in `BattleScene._createAnimations()`:
- Infantry: walk_left, walk_right, idle (green/red × normal/damage variants)
- Tank: move_left, move_right, move_up, move_down, idle (green/red × normal/damage)
- Death: dying_left, dying_right (infantry only, green/red)
- Effects: explosion, soldier_muzzle_{direction}, tank_muzzle_{direction}

Damaged animations use separate sprite sets with _damage suffix.

### Map Configuration

- Built in Tiled, exported as JSON to `assets/first-map.json`
- Hex map with staggered rows (stagger=16)
- Multiple tilesets with different tileoffsets (compensated in code)
- Layers processed in specific depth order for visual correctness

## Common Issues

**Tileset offset problems**: Red bases layer has tileoffset {x:-16, y:7} in Tiled. This is compensated in code after scaling (line 289 in BattleScene). If adding new offset tilesets, apply similar compensation.

**Tank trench movement**: Tanks can cross trenches but cannot end movement on them. The pathfinder checks `cur.costLeft > 1` before crossing and `checkLeft > 0` after. Don't remove these checks.

**Animation timing**: Movement duration is 380ms per hex step. Attack visual delays (250ms for infantry shoot) must complete before state changes to avoid sprite flickering.

**Portrait question mark**: The '?' shown before general selection uses absolute positioning. If modifying Hud layout, update both `portraitQuestion` and `portraitImage` positions.

**Depth/z-index**: Help modal uses depth 1000+, bases layer depth 1, background -1000. Other game objects at default (0).

## File Organization

```
src/
  core/          - Game logic classes (no Phaser scene dependency except Unit)
  scenes/        - Phaser scene classes
  main.js        - Phaser config and game instantiation
assets/
  first-map.json - Tiled hex map
  map/           - Tileset images
  green soldier/ - Infantry sprite frames
  red soldiers/  - Infantry sprite frames
  green tank/    - Tank sprite frames
  red tank/      - Tank sprite frames
  shoot and explode/ - Combat effect frames
  companies/     - UI icons for companies
  colonels/      - UI icons for generals
index.html       - Entry point
phaser.js        - Phaser 3 library (vendored)
```

## State Management

Game state lives in BattleScene instance variables:
- `this.units[]` - all units (including dead)
- `this.currentPlayer` - 'green' or 'red'
- `this.turnNumber` - increments when green's turn starts
- `this.selectedUnit` - currently selected unit or null
- `this.dice.value` - current dice value (0 if used)
- `this.dice.colonel` - selected general object

TurnSystem state:
- `this.initiativeDetermined` - boolean, true after first initiative roll
- `this.initiativeWinner` - 'green' or 'red', preserved for entire game
- `this.currentRound` - current round number (1-5)
- `this.currentPhase` - current phase ('initiative', 'commander', 'companies', 'actions', 'night', 'rest')
- `this.playerTurnFinished` - { green: boolean, red: boolean }, tracks who ended their turn
- `this.currentActionPlayer` - current player during actions phase

Unit state (per turn):
- `hasMoved` - boolean, true if unit moved this turn (can only move once)
- `usedStaticWeapon` - boolean, true if unit used MG/AT this turn (cannot attack after leaving)

No external state management library used.

## Coordinate System Notes

The game uses offset coordinates (q,r) with odd-row horizontal stagger. Neighbor offsets differ between odd/even rows (see HexGrid.neighbors). When adding hex-based features, use HexGrid methods rather than manual coordinate math.

World coordinates have origin at map's top-left after applying offsetX/offsetY. The map is dynamically centered and scaled based on screen size.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bahilai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
