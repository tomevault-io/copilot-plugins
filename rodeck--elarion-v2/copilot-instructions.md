## elarion-v2

> ﻿# elarion-v2 Development Guidelines

﻿# elarion-v2 Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-03-02

## Active Technologies
- TypeScript 5.x (frontend only — no backend changes) + Phaser 3.60.0, Vite 5.0.12, Google Fonts (Cinzel, Crimson Text, Rajdhani) (002-game-ui-overhaul)
- N/A — no persistence changes (002-game-ui-overhaul)
- TypeScript 5.x (frontend only — no backend changes) + Phaser 3.60.0 (game framework, texture loading, game objects, update loop), Vite 5.0.12 (static asset serving from `public/`) (004-sprite-animation)
- N/A — sprite assets are static files served from Vite's `public/` directory; no database changes (004-sprite-animation)
- TypeScript 5.x — frontend (Phaser 3, Vite) and backend (Node.js 20 LTS) + `jose` (JWT, backend), Phaser 3.60.0, `ws` library (WebSocket server) (005-session-persistence)
- `localStorage` (browser, client-side session token only); PostgreSQL 16 for all game data (005-session-persistence)
- TypeScript 5.x (frontend, backend, admin) + Phaser 3.60.0 (game frontend), Node.js 20 LTS + `ws` (backend), Express 4 (admin backend), Vite 5 (both frontends) (006-building-actions)
- PostgreSQL 16 — new `building_actions` table; `buildings` table extended with `description TEXT` (006-building-actions)
- TypeScript 5.x (frontend, backend, admin — all packages) + Phaser 3.60.0 (game frontend), Node.js 20 LTS + `ws` (game backend), Express 4 + `multer` (admin backend), Vite 5 (both frontends) (007-item-inventory)
- PostgreSQL 16 — two new tables (`item_definitions`, `inventory_items`) replacing legacy `items`/`character_items` via migration 010 (007-item-inventory)
- TypeScript 5.x (all packages — frontend, backend, shared, admin) (008-monster-combat)
- PostgreSQL 16 — new `monsters` + `monster_loot` tables; `building_actions` extended; old combat tables dropped (008-monster-combat)
- TypeScript 5.x (all packages) + Phaser 3.60 (frontend), Node.js 20 LTS + ws (backend), Express 4 (admin backend), Vite 5 (frontends) (009-squire-expeditions)
- PostgreSQL 16 — two new tables (`squires`, `squire_expeditions`), extended `building_actions` CHECK constraint (009-squire-expeditions)
- TypeScript 5.x — admin backend (Node.js 20 LTS + Express 4), admin frontend (Vite 5, vanilla TS) + Express 4, `pg` (PostgreSQL client), `node-fetch` or native `fetch` (Node 18+) for OpenRouter HTTP calls; no new npm packages required (010-ai-image-gen)
- PostgreSQL 16 (2 new tables); filesystem (same `backend/assets/` directories as existing icons) (010-ai-image-gen)
- TypeScript 5.x (frontend + backend + shared) + Phaser 3.60 (frontend game canvas), `ws` library (WebSocket), `pg` (PostgreSQL client), Vite 5 (frontend build) (011-equipment-system)
- PostgreSQL 16 — schema change via migration 014 (new column + category extension) (011-equipment-system)
- TypeScript 5.x (frontend + backend + shared) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), `pg` (PostgreSQL client), `jose` (JWT) (012-admin-commands)
- PostgreSQL 16 — no new tables; `accounts.is_admin` column already present (012-admin-commands)
- TypeScript 5.x (frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), Express 4 (admin backend), `pg` (PostgreSQL client) (013-day-night-cycle)
- PostgreSQL 16 — one new table (`map_random_encounter_tables`); no schema changes to existing tables (013-day-night-cycle)
- TypeScript 5.x (frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (game backend), Phaser 3.60 + Vite 5 (game frontend), Express 4 + `multer` (admin backend), `pg` (PostgreSQL client) (014-npc-system)
- PostgreSQL 16 — two new tables (`npcs`, `building_npcs`); filesystem for NPC icon PNGs under `backend/assets/npcs/icons/` (014-npc-system)
- TypeScript 5.x (all packages: frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), Express 4 (admin backend), `pg` (PostgreSQL client) (015-currency-system)
- PostgreSQL 16 — two new columns on existing tables (`characters.crowns`, `monsters.min_crowns`, `monsters.max_crowns`); migration file `017_currency.sql` (015-currency-system)
- TypeScript 5.x (frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), Express 4 + `multer` (admin backend), `pg` (PostgreSQL client), `jose` (JWT) (016-combat-system)
- PostgreSQL 16 — migration `018_combat_system.sql`; in-memory `Map<characterId, CombatSession>` for active sessions (016-combat-system)
- PostgreSQL 16 — new tables `crafting_recipes`, `recipe_ingredients`, `crafting_sessions`; `npcs` table extended with `is_crafter BOOLEAN` (017-crafting-system)
- TypeScript 5.x (admin backend + admin frontend) + Express 4 (admin backend), Canvas 2D API (admin frontend), `pg` (PostgreSQL client) (018-building-item-overlay)
- PostgreSQL 16 — read-only queries against existing tables (no new tables or migrations) (018-building-item-overlay)
- TypeScript 5.x (admin frontend + admin backend) + Express 4 + multer (admin backend), Canvas 2D API (admin frontend), Vite 5 (admin frontend build) (019-sprite-sheet-tool)
- PostgreSQL 16 (existing `item_definitions.icon_filename` column) + filesystem (`backend/assets/items/icons/`) (019-sprite-sheet-tool)
- PostgreSQL 16 — migration 021 (alter item_definitions, inventory_items, building_actions, characters) (020-tool-gathering)
- TypeScript 5.x (all packages: frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (game backend), Phaser 3.60 + Vite 5 (game frontend), Express 4 (admin backend), `pg` (PostgreSQL client), `jose` (JWT) (021-quest-system)
- PostgreSQL 16 — 7 new tables + 1 ALTER via migration `022_quest_system.sql` (021-quest-system)
- TypeScript 5.x (all packages: frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), Express 4 + `multer` (admin backend), `pg` (PostgreSQL client), `jose` (JWT) (022-squire-overhaul)
- PostgreSQL 16 — migration `023_squire_overhaul.sql` (new tables + ALTER); filesystem for squire icon PNGs under `backend/assets/squires/icons/` (022-squire-overhaul)
- TypeScript 5.x (frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), Express 4 (admin backend), `pg` (PostgreSQL client), `jose` (JWT) (023-player-marketplace)
- PostgreSQL 16 — migration `025_marketplace.sql` (2 new tables, 1 CHECK constraint extension) (023-player-marketplace)
- PostgreSQL 16 — migration `026_fishing_system.sql` (new tables + ALTER constraints) (024-fishing-system)
- PostgreSQL 16 — migration `027_item_disassembly.sql` (2 new tables, 3 ALTER statements) (025-item-disassembly)
- TypeScript 5.x (frontend + backend + shared) + Phaser 3.60 + Vite 5 (frontend), Node.js 20 LTS + `ws` (backend), `pg` (PostgreSQL client) (026-character-rankings)
- PostgreSQL 16 — ALTER `characters` table (add column); no new tables (026-character-rankings)
- TypeScript 5.x (frontend, backend, shared, admin) + Node.js 20 LTS + `ws` (game backend), Phaser 3.60 + Vite 5 (game frontend), Express 4 + `multer` (admin backend), `pg` (PostgreSQL client), `jose` (JWT) (027-boss-encounters)
- PostgreSQL 16 — 4 new tables (`bosses`, `boss_abilities`, `boss_loot`, `boss_instances`); in-memory `Map<bossId, BossInstance>` for active instance state (027-boss-encounters)
- TypeScript 5.x (all packages) + Node.js 20 LTS + `ws` (backend), Phaser 3.60 + Vite 5 (frontend), Express 4 (admin backend), `pg` (PostgreSQL client), `jose` (JWT) (029-arena-system)
- PostgreSQL 16 — 3 new tables (`arenas`, `arena_monsters`, `arena_participants`), 1 ALTER (`characters.arena_id`); in-memory `Map<string, PvpCombatSession>` for active fights (029-arena-system)
- PostgreSQL 16 — ALTER `characters` table (6 new columns), ALTER `npcs` table (1 new column); migration `033_stat_allocation.sql` (030-stat-allocation)
- PostgreSQL 16 — migration `034_stat_training.sql` (new table + ALTER) (031-stat-training)
- TypeScript 5.x (frontend, backend, shared, admin) + Phaser 3.60 (frontend), Node.js 20 LTS + ws (backend), Express 4 (admin backend), pg (PostgreSQL client), jose (JWT), Vite 5 (frontends) (032-skill-development)
- PostgreSQL 16 — migration `035_skill_development.sql` (new tables + ALTER) (032-skill-development)
- TypeScript 5.x (frontend + backend + shared + admin) + Phaser 3.60 (frontend), Node.js 20 LTS + `ws` (backend), Express 4 (admin backend), Vite 5 (frontends) (033-weapon-attributes)
- PostgreSQL 16 — migration 037 (ALTER `item_definitions`) (033-weapon-attributes)
- TypeScript 5.x (frontend + backend + shared) + Phaser 3.60 (frontend), Node.js 20 LTS + `ws` (backend), Express 4 (admin backend), Vite 5 (frontends) (034-item-variation)
- PostgreSQL 16 — migration 038 (ALTER `inventory_items`, ALTER `marketplace_listings`) (034-item-variation)
- TypeScript 5.x (frontend, backend, shared, admin) + Phaser 3.60 (frontend), Node.js 20 LTS + `ws` (backend), Express 4 (admin backend), Vite 5 (frontends) (035-combat-fatigue)
- PostgreSQL 16 — new `fatigue_config` table (migration 039); in-memory `FatigueState` on combat session objects (035-combat-fatigue)
- PostgreSQL 16 — two new tables (`warehouse_slots`, `warehouse_items`); `building_actions` CHECK constraint extended (036-warehouse-system)
- TypeScript 5.x (frontend only) + Phaser 3.60.0 (game framework), Vite 5 (dev server) (037-player-interaction)
- N/A — no persistence changes; all state is in-memory on frontend (037-player-interaction)
- PostgreSQL 16 — migration `042_energy_system.sql` (ALTER `characters`, ALTER `building_actions` config) (038-energy-system)
- TypeScript 5.x (frontend + backend + shared + admin) + Phaser 3.60 (frontend), Node.js 20 LTS + `ws` (backend), Express 4 + `multer` (admin backend), Vite 5 (frontends) (039-spell-system)
- PostgreSQL 16 — migration 043 (5 new tables, 2 ALTER statements) (039-spell-system)

- TypeScript 5.x — used on both frontend and backend. (001-game-design)

- TypeScript 5.x + Express + multer (admin backend), HTML5 Canvas 2D (map editor), PostgreSQL 16 (003-city-map-system)

## Project Structure

```text
backend/
frontend/
shared/
admin/
  backend/   # Express REST API for map editor (port 4001)
  frontend/  # Vite + Canvas 2D map editor UI
```

## Commands

npm test && npm run lint

## Code Style

TypeScript 5.x — used on both frontend and backend.: Follow standard conventions

## Recent Changes
- 039-spell-system: Added TypeScript 5.x (frontend + backend + shared + admin) + Phaser 3.60 (frontend), Node.js 20 LTS + `ws` (backend), Express 4 + `multer` (admin backend), Vite 5 (frontends)
- 038-energy-system: Added TypeScript 5.x (frontend + backend + shared + admin) + Phaser 3.60 (frontend), Node.js 20 LTS + `ws` (backend), Express 4 (admin backend), Vite 5 (frontends)
- 037-player-interaction: Added TypeScript 5.x (frontend only) + Phaser 3.60.0 (game framework), Vite 5 (dev server)



<!-- MANUAL ADDITIONS START -->

## Adding a New Building Action Type

When introducing a new `action_type` (e.g., `'fishing'`, `'crafting_station'`), you must update **all 7 locations** listed below. Missing any one of them causes silent failures — the action either won't save, won't render, or will display as a broken travel action.

### Checklist

1. **DB CHECK constraint** — Migration file extending `building_actions.action_type`:
   ```sql
   ALTER TABLE building_actions DROP CONSTRAINT building_actions_action_type_check;
   ALTER TABLE building_actions ADD CONSTRAINT building_actions_action_type_check
     CHECK (action_type IN ('travel', 'explore', 'expedition', 'gather', 'marketplace', '<NEW_TYPE>'));
   ```

2. **Shared protocol types** — `shared/protocol/index.ts`:
   - Create a `<NewType>BuildingActionDto` interface
   - Add it to the `BuildingActionDto` union type
   - Add `'<new_type>'` to `CityBuildingActionPayload.action_type` union

3. **Game backend city-map-loader** — `backend/src/game/world/city-map-loader.ts`:
   - Add an `if (a.action_type === '<new_type>')` branch in the `.map()` callback that builds `BuildingActionDto` objects
   - **Without this, the action renders as "Travel to Zone undefined"**

4. **Game backend building-action-handler** — `backend/src/game/world/building-action-handler.ts`:
   - Add an `if (action.action_type === '<new_type>')` branch in the action dispatch logic

5. **Admin backend buildings route** — `admin/backend/src/routes/buildings.ts`:
   - Add `'<new_type>'` to the action_type validation check (~line 265)
   - Add an `else if (action_type === '<new_type>')` branch in the config processing logic for the create endpoint
   - **Without the config branch, the action falls through to expedition validation and fails with "base_gold must be a non-negative integer"**

6. **Admin frontend API types** — `admin/frontend/src/editor/api.ts`:
   - Add `'<new_type>'` to `BuildingAction.action_type` union
   - Add `'<new_type>'` to `createBuildingAction` parameter's `action_type` union

7. **Admin frontend properties panel** — `admin/frontend/src/ui/properties.ts`:
   - Add `['<new_type>', 'Label']` to the `actionTypes` array (dropdown options)
   - Add a fields `<div>` for the new type's config (if any)
   - Add show/hide logic in the `typeSelect` change handler
   - Add a save handler branch in the save button click handler
   - Add a display label branch in the action list renderer

### Also consider

- **Game frontend BuildingPanel** — `frontend/src/ui/BuildingPanel.ts`: Add rendering for the new action type button/UI
- **Game frontend GameScene** — `frontend/src/scenes/GameScene.ts`: Add message handlers if the action type uses custom WebSocket messages
- **game-entities skill** — `.claude/commands/game-entities.md`: Document the new action type's config format
- **game-entities script** — `scripts/game-entities.js`: Update `VALID_ACTION_TYPES` array

## Adding a New Equipment Slot

When adding a new equipment slot (e.g., `'ring'`, `'amulet'`), update these locations:

1. **DB migration** — Extend `inventory_items.equipped_slot` CHECK constraint and `item_definitions.category` CHECK constraint
2. **Shared protocol** — `shared/protocol/index.ts`: Add to `ItemCategory`, `EquipSlot`, and `EquipmentSlotsDto`
3. **Equipment handler** — `backend/src/game/equipment/equipment-handler.ts`: Add to `VALID_SLOTS` array
4. **Equipment queries** — `backend/src/db/queries/equipment.ts`: Add to `SLOT_CATEGORY_MAP` and the `slots` object initializer in `getEquipmentState()`
5. **Frontend equipment UI** — `frontend/src/ui/EquipmentPanel.ts`: Add slot to grid layout and `SLOT_CONFIGS`
6. **Frontend left panel** — `frontend/src/ui/LeftPanel.ts`: Add category to `EQUIPPABLE_CATEGORIES`
7. **Admin frontend** — `admin/frontend/src/editor/api.ts`: Update `VALID_CATEGORIES` if the script validates categories

Combat stats (`computeCombatStats` in `combat-stats-service.ts`) and effective stats queries aggregate ALL equipped items generically — no changes needed there.

## Adding a New Spell

When adding a new spell definition:

1. **Admin panel** — Spell Manager → Create: Set name, description, effect_type, base effect_value, base duration_seconds, icon
2. **Spell levels** — Admin panel → Edit spell → Levels tab: Set per-level effect_value, duration_seconds, gold_cost (levels 1–5)
3. **Spell costs** — Admin panel → Edit spell → Costs tab: Set per-level item costs (item_def_id + quantity, multiple items per level)
4. **Spell book item** — Create item with category `spell_book_spell`, link to spell via `spell_id`

Key files for the spell system:
- **DB tables** — `spells`, `spell_levels`, `spell_costs`, `character_spells`, `active_spell_buffs` (migration 043)
- **DB queries** — `backend/src/db/queries/spells.ts`, `backend/src/db/queries/spell-progress.ts`, `backend/src/db/queries/spell-buffs.ts`
- **Backend handlers** — `backend/src/game/spell/spell-cast-handler.ts` (WS: `spell.cast`, `spell.cast_on_player`), `backend/src/game/spell/spell-book-handler.ts` (WS: `spell-book-spell.use`), `backend/src/game/spell/spell-state-handler.ts` (WS: `spell.request_state`)
- **Buff service** — `backend/src/game/spell/spell-buff-service.ts` (stat modifiers for `computeCombatStats`)
- **Shared protocol** — `shared/protocol/index.ts`: Spell DTOs, payloads, message types
- **Frontend** — `frontend/src/ui/SpellPanel.ts`, `frontend/src/ui/SpellDetailModal.ts`, `frontend/src/ui/BuffBar.ts`
- **Admin routes** — `admin/backend/src/routes/spells.ts` (CRUD)
- **Admin UI** — `admin/frontend/src/ui/spell-manager.ts`
- **Admin commands** — `/spells.all <player>` (grant all spells), `/abilities.all <player>` (renamed from `/skill_all`)
- **Scripts** — `scripts/game-data.js spells`, `scripts/game-data.js spell-buffs`

## Adding a New Skill Book

When adding a new skill book (linking a consumable item to an ability for skill leveling):

1. **Create the skill book item** via `create-skill-book` with `name`, `description`, `stack_size`, and `ability_id`
2. **Define level stats** via `set-ability-levels` with `ability_id` and `levels` array (5 levels with effect_value, mana_cost, duration_turns, cooldown_turns)
3. **Add to boss loot tables** via admin panel or `game-entities` script
4. **Add to expedition rewards** in high-tier expedition building action configs

Key files for the skill development system:
- **DB tables** — `ability_levels`, `character_ability_progress` (migration 035)
- **DB queries** — `backend/src/db/queries/ability-levels.ts`, `backend/src/db/queries/ability-progress.ts`
- **Backend handler** — `backend/src/game/skill/skill-book-handler.ts` (WS: `skill-book.use`)
- **Shared protocol** — `shared/protocol/index.ts`: `SkillBookUsePayload`, `SkillBookResultPayload`, `SkillBookErrorPayload`, `AbilityLevelStatsDto`
- **Frontend modal** — `frontend/src/ui/SkillDetailModal.ts`
- **Frontend loadout** — `frontend/src/ui/LoadoutPanel.ts` (level badges, progress bars, cooldown timers)
- **Admin routes** — `admin/backend/src/routes/abilities.ts` (GET/PUT /:id/levels)
- **Admin UI** — `admin/frontend/src/ui/ability-manager.ts` (modal with level stats editor)
- **Scripts** — `scripts/game-data.js ability-levels`, `scripts/game-entities.js create-skill-book`, `scripts/game-entities.js set-ability-levels`

## Adding a New Item Category

Update these locations:

1. **DB migration** — Extend `item_definitions.category` CHECK constraint
2. **Shared protocol** — `shared/protocol/index.ts`: Add to `ItemCategory` type
3. **game-entities script** — `scripts/game-entities.js`: Add to `VALID_CATEGORIES` array
4. **game-entities skill** — `.claude/commands/game-entities.md`: Update `create-item` docs
5. **Admin frontend** — If category is stackable, check `STACKABLE_CATEGORIES` in the script

Existing categories: `resource`, `food`, `heal`, `weapon`, `boots`, `shield`, `greaves`, `bracer`, `tool`, `helmet`, `chestplate`, `ring`, `amulet`, `skill_book`.

## Adding a New Quest Reward Type

Update these locations:

1. **Shared protocol** — `shared/protocol/index.ts`: Add to `RewardType` type
2. **Quest service** — `backend/src/game/quest/quest-service.ts`: Add case in `grantQuestRewards()` switch
3. **Admin backend quest routes** — `admin/backend/src/routes/quests.ts`: Add to `VALID_REWARD_TYPES` array
4. **Admin frontend quest manager** — `admin/frontend/src/ui/quest-manager.ts`: Add to `REWARD_TYPES` array and add fields case in `buildRewardSpecificFields()`
5. **game-entities script** — `scripts/game-entities.js`: Add to `VALID_REWARD_TYPES` array
6. **game-entities skill** — `.claude/commands/game-entities.md`: Document the new reward type in `create-quest` rewards docs

The `resolveRewardTarget()` function returns `{ name: null, icon_url: null }` for unknown types — this is fine for currency-like rewards (xp, crowns, points).

## Adding a New NPC Role

When adding a new NPC boolean flag (e.g., `is_disassembler`, `is_merchant`), update these locations:

1. **DB migration** — `ALTER TABLE npcs ADD COLUMN is_<role> BOOLEAN NOT NULL DEFAULT false`
2. **Shared protocol** — `shared/protocol/index.ts`: Add `is_<role>: boolean` to `NpcDto`
3. **Game backend NPC queries** — `backend/src/db/queries/npcs.ts`: Add to `Npc`, `BuildingNpc`, `ZoneNpcRow` interfaces and SELECT column lists in `getNpcsForBuilding()` and `getNpcsForZone()`
4. **Game backend city-map-loader** — `backend/src/game/world/city-map-loader.ts`: Add `is_<role>: n.is_<role> ?? false` to NPC DTO mapping
5. **Game frontend BuildingPanel** — `frontend/src/ui/BuildingPanel.ts`: Add dialog option in `renderNpcPanel()` with `if (npc.is_<role>)` check, add callback type + setter + field
6. **Game frontend GameScene** — `frontend/src/scenes/GameScene.ts`: Wire the callback via `buildingPanel.setOn<Role>Open()`
7. **Admin backend NPC routes** — `admin/backend/src/routes/npcs.ts`: Add to `npcToResponse()`, add `PUT /:id/<role>` toggle endpoint
8. **Admin frontend API** — `admin/frontend/src/editor/api.ts`: Add `toggleNpc<Role>()` function
9. **Admin frontend NPC manager** — `admin/frontend/src/ui/npc-manager.ts`: Add checkbox + change handler
10. **gd.design skill** — `.claude/commands/gd.design.md`: Add column to NPC table template

Existing NPC roles: `is_crafter`, `is_quest_giver`, `is_squire_dismisser`, `is_disassembler`, `is_trainer`.

## Adding a New Stat Training Item

When adding a new training item (maps a consumable item to a stat for NPC training), use the admin API or game-entities script:

1. **Create the item** via `create-item` (category: resource, stackable)
2. **Map as training item** via `create-stat-training-item` with `item_def_id`, `stat_name`, `tier` (1-3), `base_chance` (1-100), `decay_per_level`, `npc_id`
3. **Ensure NPC has `trainer_stat` set** via `set-npc-trainer-stat` with the stat name

Key files for the stat training system:
- **DB table** — `stat_training_items` (migration 034)
- **DB queries** — `backend/src/db/queries/stat-training.ts`
- **Backend handler** — `backend/src/game/training/stat-training-handler.ts` (WS: `stat-training.open`, `stat-training.attempt`)
- **Shared protocol** — `shared/protocol/index.ts`: `StatTrainingOpenPayload`, `StatTrainingAttemptPayload`, `StatTrainingStatePayload`, `StatTrainingResultPayload`, `StatTrainingItemDto`
- **NPC field** — `npcs.trainer_stat` (which stat the NPC trains; null = not a stat trainer)
- **Frontend modal** — `frontend/src/ui/StatTrainingModal.ts`
- **Admin route** — `admin/backend/src/routes/stat-training.ts` (CRUD)
- **Scripts** — `scripts/game-data.js stat-training`, `scripts/game-entities.js create-stat-training-item`

Success formula: `max(5, base_chance - character_level * decay_per_level)`. Cap: `10 * (level - 1)` per stat (shared with manual allocation).

## Adding a New Tool Type

When adding a new `tool_type` value (e.g., `'kiln'`), update these locations:

1. **DB migration** — Extend `item_definitions.tool_type` CHECK constraint
2. **Admin backend items route** — `admin/backend/src/routes/items.ts`: Add to `VALID_TOOL_TYPES` array
3. **Admin frontend item modal** — `admin/frontend/src/ui/item-modal.ts`: Add `<option>` to tool type select
4. **game-entities script** — `scripts/game-entities.js`: Add to `VALID_TOOL_TYPES` array
5. **game-entities skill** — `.claude/commands/game-entities.md`: Document the new tool type in `create-item` docs

Existing tool types: `'pickaxe'`, `'axe'`, `'fishing_rod'`, `'kiln'`.

## Adding a New Boss

When creating a new boss, configure these through the **admin panel** (no code changes needed):

1. **Boss definition** — Admin panel → Boss Manager → Create: Set name, description, stats (HP, ATK, DEF), XP, crowns, building assignment, respawn time range, active toggle
2. **Boss abilities** — Admin panel → Boss Manager → Edit boss → Abilities tab: Assign existing abilities with priority ordering (higher priority fires first)
3. **Boss loot** — Admin panel → Boss Manager → Edit boss → Loot tab: Add items with drop chance (%) and quantity
4. **Boss icon** — Admin panel → Boss Manager → Edit boss → Upload icon: Combat icon PNG (stored in `backend/assets/bosses/icons/`)
5. **Boss sprite** — Admin panel → Boss Manager → Edit boss → Upload sprite: Map sprite PNG (stored in `backend/assets/bosses/sprites/`)
6. **Verify instance spawned** — Check `node scripts/game-data.js boss-instances` to confirm the boss instance is alive at the assigned building

### If adding boss-related code changes (new boss features):

1. **DB queries** — `backend/src/db/queries/bosses.ts`: Add new query functions
2. **Instance manager** — `backend/src/game/boss/boss-instance-manager.ts`: Update lifecycle logic
3. **Combat handler** — `backend/src/game/boss/boss-combat-handler.ts`: Update combat flow
4. **Shared protocol** — `shared/protocol/index.ts`: Add/modify boss message types
5. **Frontend** — `frontend/src/ui/CombatScreen.ts`, `frontend/src/entities/BossSprite.ts`, `frontend/src/ui/BossInfoPanel.ts`
6. **Admin routes** — `admin/backend/src/routes/bosses.ts`: Update admin API
7. **Admin UI** — `admin/frontend/src/ui/boss-manager.ts`: Update admin panel
8. **game-data script** — `scripts/game-data.js`: Update `bosses` and `boss-instances` commands
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rodeck)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rodeck)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
