## the-lords-ledger

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

The Lord's Ledger is a medieval economic simulation game for 6th graders. Players inherit a feudal estate and manage it for 40 turns (10 years x 4 seasons). Built with React 19 + Vite 7 + Tailwind CSS 4, pure JavaScript (no TypeScript).

## Commands

```bash
npm run dev       # Vite dev server at http://localhost:5173
npm run build     # Production build → dist/
npm run lint      # ESLint (flat config, React hooks + refresh rules)
npm run preview   # Preview production build
```

No test framework is configured. Playwright is installed but no test suite exists.

## Architecture

### State Machine (useReducer)

All game state lives in a single `useReducer` in `App.jsx`. The reducer (`src/engine/gameReducer.js`, 2600+ lines, 70+ action types) is the source of truth for every state transition.

**Phase flow:**
```
title → management → seasonal_action → seasonal_resolve → random_event → random_resolve → management (next turn)
                                                                                          ↗
game_over (population = 0, or bankrupt 3 turns)
victory (survived 40 turns)
```

During `management`, the player browses 9 tabs and makes decisions. Clicking "Simulate Season" runs the economy engine, processes military morale/upkeep, checks for raids, then fires events.

### Engine Layer (`src/engine/`)

All engine files export **pure functions only** — no side effects, no I/O, no DOM access. Math.random() is the only exception (intentional RNG for event selection).

- **gameReducer.js** — 70+ action types. Event data injected via action payloads. Contains all state transitions for every subsystem.
- **economyEngine.js** — `simulateEconomy(state)` runs: production → levy labor penalty → consumption → upkeep (typed garrison) → tax (Autumn only) → passive income → population growth.
- **raidEngine.js** — `resolveRaid(raidType, defenseRating, defenseThreshold, garrison, castleLevel, inventory)`. Defense rating system (not raw garrison count).
- **flipEngine.js** — Perspective flips (Serf, Merchant, Noble, Knight) and CYOA branching narratives.
- **synergyEngine.js** — `checkSynergies()` evaluates 10+ strategy paths with 3 tiers each. Provides passive income, trade bonuses, population growth.
- **meterUtils.js** — Legacy name. Contains `translateEffects()` (converts old meter-format event effects to resource deltas), `applyResourceEffects()`, `checkGameOver()`.
- **eventSelector.js** — `selectSeasonalEvent()`, `selectRandomEvent()` with turn-based gating.

### Resource System

Concrete resources (no abstract meters at the top level):
- **Denarii** — money for building, trading, paying soldiers
- **Food** — grain + livestock + fish in inventory, consumed by population each season
- **Population** — families on the estate
- **Garrison** — total soldier count (backward-compat number synced from `state.military.garrison`)
- **Morale** — military morale (0-100), affects defense rating and desertion

Game over: population = 0 (depopulation) or denarii = 0 for 3+ consecutive turns (bankruptcy).

### Tabs (9 total, all unlocked from turn 1)

`estate`, `map`, `market`, `military`, `people`, `hall` (Great Hall), `chapel`, `forge` (Blacksmith), `chronicle`

### Component Patterns

**Complex tabs** receive `state` and `dispatch` directly — they manage their own sub-interactions:
- GreatHall, ChapelTab, BlacksmithTab, Tavern, Watchtower, PeopleTab, MarketSquare

**Simple tabs** receive handler functions from App.jsx:
- EstateTab (`onBuild, onDemolish, onRepair, onUpgrade`)
- MilitaryTab (`onRecruit, onDismiss, onUpgradeFortification, onOpenWatchtower`)

### Key Subsystems

**Military** (`state.military`):
- Typed garrison: `{ levy, menAtArms, knights }` with different costs/defense values
- Three fortification tracks: walls (0-4), gate (0-4), moat (0-3) with prerequisites
- Morale (0-100): affects defense multiplier (×0.70 to ×1.25), triggers desertion below 20
- Defense rating = (garrison defense + fortification defense) × morale modifier + watchtower bonus
- Raid thresholds: criminal = 25, scottish = 50
- Top-level `state.garrison` kept as total count for backward compat (Dashboard, economyEngine)
- `state.castleLevel` synced to walls level (for passive income)

**Great Hall** (`state.greatHall`):
- Disputes (16), audiences (20), decrees (10), council topics (4), feasts
- Edmund the Steward: context-aware dialogue (30+ lines), trust mechanic (0-100), mood tied to treasury
- Reputation: 6 tracks (merciful/stern/wealthy/pious/militant/balanced) with 3 title tiers each
- Sub-components: DisputeScreen, AudienceChamber, DecreeDesk, CouncilChamber, FeastHall

**Blacksmith** (`state.blacksmith`):
- Godric NPC with respect/mood system and dialogue
- ForgingGame component (forge state is local useState — Phase 2 will move to reducer)
- Supply events, iron vein investment, Mortimer plot thread

**Chapel** (`state.chapel`):
- Faith/piety meters, Anselm & Caedmon NPCs
- Tithe system, moral dilemmas, manuscript copying minigame

**Market** (`state.market`):
- Haggle system with counter-offers, merchant reputation tiers
- Local merchants + foreign traders, seasonal market events

**Raids** (`state.raids`):
- Criminal and Scottish raiders with defense comparison
- RaidScreen overlay: warning phase → defend → result
- `resolveRaid()` uses defense rating vs threshold (not raw garrison)

**Synergies** (`state.synergies`):
- 10+ strategy paths × 3 tiers (Wool Baron, Breadbasket, Fortress, etc.)
- Checked each turn via `checkSynergies()`, provide passive bonuses
- Tier 3 unlocks unique victory titles

### Data Layer (`src/data/`)

20 data files. All export constants and pure functions. Key files:
- **buildings.js** — 20+ buildings with costs, production, upkeep, synergies, upgrade paths, land plots
- **military.js** — Soldier types, fortification tracks, morale levels, defense calculation helpers, tooltips, scribe's notes
- **economy.js** — Resource types, prices, tax rates, castle levels, season multipliers, constants
- **greatHall.js** — Edmund dialogue matrix, reputation system, trust/mood mechanics
- **disputes.js** / **audience.js** / **decrees.js** — Content for Great Hall interactions
- **market.js** — Merchants, haggle config, reputation tiers, market events
- **endings.js** — Victory titles, failure narratives, `victorySummary(state)`
- **seasonalEvents.js** / **randomEvents.js** — Event pools with effects and scribe's notes

### Estate System

- `state.buildings`: array of `{ instanceId, type, condition, builtOnTurn }` objects
- Land system: `state.totalPlots` (default 12), each building costs 1-2 plots
- Condition: degrades 5-10%/season (×1.5 winter), affects output (Good ×1.0, Fair ×0.75, Poor ×0.5, Ruined ×0)
- Seasonal farm multipliers: Spring ×0.5, Summer ×1.0, Autumn ×1.5, Winter ×0.25
- Building synergies in `buildingSynergies` array in buildings.js

## Key Conventions

**State immutability:** Reducer always returns `{ ...state, field: newValue }`. Never mutate in place.

**Action shape:** `{ type: "SCREAMING_SNAKE", payload: { ... } }`. Event data arrays passed via payload, not imported by reducer.

**Chronicle entries:** `{ text, season, year, turn, type: "system"|"action"|"event" }`.

**Theming:** Royal Dark medieval aesthetic. Custom CSS variables in `src/index.css` via Tailwind's `@theme`. Fonts: Cinzel (headings), Cinzel Decorative (display), Crimson Text (body), Almendra (NPC flavor text).

**Styling approach:** Inline `style={{}}` for theme colors, Tailwind utility classes for layout. This is intentional — the custom parchment palette doesn't map cleanly to Tailwind color utilities.

**Lint rules:** ESLint flat config. Unused vars must match `/^[A-Z_]/u`. No `Math.random()` in render — use pre-computed arrays or `useState(() => ...)` initializer.

## Adding Features

1. Add data definitions to `src/data/` (buildings, events, constants)
2. Add pure logic to `src/engine/` (economy calculations, validation)
3. Add reducer action case to `gameReducer.js`
4. Add or update component in `src/components/`
5. Wire dispatch handler in `App.jsx` (simple tabs) or pass `dispatch` directly (complex tabs)

## Legacy / Migration Notes

- Old `defenseUpgrades` array kept but deprecated (INSTALL_DEFENSE is no-op)
- Old UPGRADE_CASTLE is no-op — use UPGRADE_FORTIFICATION with `{ track }` payload
- RECRUIT_SOLDIERS / DISMISS_SOLDIERS take `{ soldierType, count }` payload (default: "levy")
- Events still use old effect format `{ treasury: N, people: N }` translated via `translateEffects()` in meterUtils.js
- `getBuildingType(b)` adapter in economyEngine.js handles both string and object building formats

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goldbar123467)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/goldbar123467)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
