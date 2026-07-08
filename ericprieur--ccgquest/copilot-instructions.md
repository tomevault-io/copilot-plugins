## ccgquest

> This file is auto-loaded into Claude's context. Keep it short and focused on

# Project notes for Claude

This file is auto-loaded into Claude's context. Keep it short and focused on
things that aren't obvious from the code alone.

## Codex (debug `C` key)

The codex is mostly **auto-generated** from the same data the game uses at
runtime — adding a card / enemy / loot table to the right registry usually
makes it appear in the codex with no extra work. The cache is built lazily
on first codex access and stored in module-level `_codexSourceCache` in
`src/main.js`. A page reload refreshes it.

### What auto-discovers (do nothing extra)

| You added… | And it appears in the codex via… |
|---|---|
| A card creator in `CARD_REGISTRY` (main.js) | Player Cards tab + correct subtype filter |
| A card to a starter deck function (cards.js) | Starter source line + Decks tab section |
| A card to a class ability-choice list (cards.js) | Lost Shrine source line (if `tier === 1`) |
| An entry in `LOOT_TABLES` (main.js) | Loot Tables tab section + per-card `Loot: X (n%)` source line |
| An entry in `SHOP_INVENTORIES` (main.js) | `Shop: <Name> (<price>g)` source line |
| `lootCards: ['x']` on an encounter (encounter.js) | `Drop: <Encounter Name>` source line |
| `previewCreature: ...` on a card (cards.js) | Summons tab entry, side stamped `player` |
| `enemy.addCreature(new Creature(...))` in `setupEnemyForCombat` | Summons tab entry, side stamped `enemy` |
| A new card in an enemy's `enemy.deck.addCard(...)` calls | Enemy Cards tab + Decks tab section + `Enemy: X` source line |

### What needs a manual touch

When adding any of these, also update the listed file:

1. **A new enemy id** in `setupEnemyForCombat` (main.js ~2009) →
   - Add the id to `enemyIds = [...]` in `buildCodexSourceCache`
     (main.js ~12517) so the sandbox scan picks it up.
   - If you want a portrait under **Heroes & Monsters**, add the id to
     `getCodexMonsterIds()` (main.js ~12260) **and** ensure
     `assets/Cards/<EnemyArt>.jpg` is loaded as `creature_<id>` in
     `loadAssets()`.
2. **A new shop** in `SHOP_INVENTORIES` →
   - Add a friendly label to `shopLabels = {...}` in
     `buildCodexSourceCache` (main.js ~12480), otherwise the source line
     falls back to a title-cased id.
3. **A new loot table** in `LOOT_TABLES` →
   - Optional but recommended: add to `LOOT_TABLE_LABELS` and
     `LOOT_TABLE_NOTES` (main.js ~371) for a friendly title and the
     one-line description shown under the section header.
4. **A new non-persistent player card** (token-like, never in
   `CARD_REGISTRY`) → add the creator to `ALL_EXTRA_CARD_CREATORS`
   (main.js ~11492). Currently used for `Goodberry` and the four
   power-choice cards (`Fire`, `Ice`, `Feline Form`, `Bear Form`).
5. **A new power (player or enemy)** → add the creator to
   `ALL_POWER_CREATORS` (main.js, near the codex section). Without this
   the power gets *source lines* but no codex entry. Then:
   - If it's a **player class power**, also add the id to
     `PLAYER_POWER_IDS` so the codex tags it player-side (default is
     enemy).
6. **A sound fired imperatively from an event** (any
   `playSound('foo', ...)` call NOT routed through
   `CARD_SFX_OVERRIDES`/play/flesh/blocked/defense — e.g. a card's
   `playCardAmbient` special-case, an enemy power's start-of-turn
   hook, a fight-start splash, a death cue, an ambient layer) →
   - **The common case needs NO codex wiring:** a card's own cast / hit
     sound added via `CARD_SFX_OVERRIDES` (`play` / `flesh` / `blocked`)
     is surfaced in the codex Sounds tab automatically — the builder
     iterates `CARD_SFX_OVERRIDES` and lists the card under its sound.
     You ONLY have to add the alias to `SOUND_MAP`. The bullets below
     cover sounds that fall OUTSIDE that auto-surfaced path.
   - Make sure the alias is in `SOUND_MAP` (`src/sound.js`); if it's
     only in `SOUND_PACKS` it won't actually play.
   - If the cue is layered on top of a card cast, register it in
     `CARD_SFX_HINTS` (main.js, near `CARD_SFX_OVERRIDES`) so the
     codex Sounds tab shows that card under the file.
   - For per-enemy fight-start / fight-end cues, return the alias
     from `getFightStartSfxKey` / `getDeathSfxKey`. The codex
     Character panel reads those.
   - For looping `playAmbienceLayer(...)` beds, add the file path
     to `MUSIC_TAGS` with a `combat ambience: <fight name>` tag so
     the codex Sounds tab surfaces the wiring.
   - Per-creature swing sounds (Magma Mephit fire whoosh, Shark
     splash, etc.) belong in the creature-name branches of
     `getWeaponSfxKeys` (`flesh`/`blocked`/`play` keys).
7. **A new player summon that no card has `previewCreature` for**
   (e.g. spawned only by an effect handler) → add an explicit
   `addCreature(new Creature({...}), 'Summoned by: <source>')` block in
   `buildCodexSourceCache` (main.js, near the existing Restless Bone /
   Thorb additions). Set:
   - `_codexSide = 'player'` (Decks/Summons side filter)
   - `_sourceRarity = '<rarity of source card>'` (drives frame asset —
     `uncommon`+ uses the ornate frame)
   - `_sourceSubtype = '<subtype of source card>'` (drives frame tint —
     `'ability'` = purple, `'allies'` = brown, etc., via `SUBTYPE_COLORS`)

   For creatures from a card's `previewCreature`, all three fields are
   stamped automatically from the parent card by `buildCodexSourceCache`
   and by the side-preview call sites in `drawHoverPreview`,
   `drawClassCard`, and the loot-modal preview.

### Source-line linking

Source entries in the right-side stats panel are objects
`{ text, link? }`. `link` is optional metadata for click-to-navigate.
Currently supported link types:

- `{ type: 'loot', id: '<table_id>' }` — jumps to the Loot Tables tab and
  highlights that table.
- `{ type: 'deck', id: '<deck_id>' }` — jumps to the Decks tab and
  highlights that deck. Deck ids: `starter_<class>` for starter decks,
  `enemy_<enemyId>` for monster decks.

When adding a new source category that should be navigable, follow the
same pattern: add the `link` field where the source is recorded in
`buildCodexSourceCache`, then handle the `link.type` in the `goto-link`
case of `handleCodexClick`.

### Sanity check after a content change

1. Reload the page (cache is module-level and lazy).
2. Press `C` (debug mode required).
3. Check Player or Enemy tab for the new card / power / creature.
4. Click it and verify the right-side **Sources** list looks correct
   (class restriction, starter, ability choice, loot, drop, shop,
   enemy, etc.).
5. If it's a deck or loot member, follow the blue link in Sources to
   confirm the navigation lands on the right section.

### Lore tab — hand-authored, NOT auto-discovered

The codex **Lore** tab (`drawCodexLoreGrid` in `src/main.js`) is the one
codex section that is *not* generated from runtime data. Its entries live
in **`src/lore.js`** as the `LORE_ENTRIES` array — each is
`{ name, type, description }` where `type` is one of `LORE_TYPES`
(`Person` / `Place` / `Faction` / `Creature` / `Artifact` / `Event` /
`Deity`). To add lore, just append an object; the tab filters by type
pill and searches across name + type + description automatically. New
category? Add it to `LORE_TYPES` and give it a tint in
`LORE_TYPE_COLORS`. Descriptions should stay ~140 chars (wraps to 2
lines) and only state what the narrative text actually establishes.

## Image assets — JPG by default, PNG only for transparency

Card art, backgrounds, and map art are stored as **JPG** (quality 4 via
`ffmpeg-static`). Source PNGs from Midjourney drop into `public/assets/`
and get bulk-converted in place by:

```
npm run png-to-jpg                           # whole public/assets tree
npm run png-to-jpg -- --keep                 # don't unlink originals
npm run png-to-jpg -- --quality 3            # higher quality, larger
npm run png-to-jpg public/assets/Cards       # subtree
```

The script (`scripts/convert-png-to-jpg.mjs`) **skips any path whose
directory chain contains a segment in `SKIP_DIR_SEGMENTS`** — currently
just `Icons`. Icons (status icons, frames, inline pills, banners,
buttons) must stay PNG because their alpha channel is composited over
card art; JPG would flatten transparency to a solid black box.

If you add a new directory whose images need alpha (UI overlays, etc.),
add its directory name to `SKIP_DIR_SEGMENTS` in the script **before**
running it. Anything outside the skip list is fair game.

When you wire a new image into code, reference it as `.jpg` unless it
lives under `assets/Icons/` (then `.png`). After running the script,
`git status` will show the `.png` deletions next to the matching `.jpg`
adds — commit both in the same commit so the working tree stays in sync.

### Source filename convention — CamelCase, no spaces

The converter **preserves the filename** and only swaps the extension
(`Bloody Eye Patch.png` → `Bloody Eye Patch.jpg`). `CARD_ART_MAP` keys
on the exact filename, so source PNGs straight off Midjourney
(`Bloody Eye Patch.png`, `Kraken's Eye Spyglass.png`) need to be
**renamed before conversion**:

- Strip spaces, apostrophes, and dashes.
- CamelCase each word (`BloodyEyePatch.png`, `KrakensEyeSpyglass.png`).
- Match what you put in `CARD_ART_MAP` exactly.

If you wire `CARD_ART_MAP[my_card] = 'MyCardArt.jpg'` but the file on
disk is still `My Card Art.png`, the converter happily produces
`My Card Art.jpg` and the card renders as a brown placeholder. Rename
the PNG first, **then** run `npm run png-to-jpg`.

### End-to-end checklist for new card art

1. Drop the source PNG into `public/assets/Cards/` (or the appropriate
   subdir).
2. Rename to CamelCase (`BloodyEyePatch.png`) — no spaces, no
   apostrophes, no dashes.
3. Run `npm run png-to-jpg public/assets/Cards`.
4. Add the entry to `CARD_ART_MAP` in `src/card-art.js`
   (`bloody_eye_patch: 'BloodyEyePatch.jpg'`).
5. `git status` should show one `.png` deletion + one `.jpg` add per
   image — commit both together.

## Map backgrounds — register in BOTH the map data AND the asset loader

A new map area's image needs **two** registrations:

1. **`map.mapImages = { <mapArea>: 'Maps/<File>.jpg' }`** in the
   `createXxxMap` function (`src/map.js`). This is what the map view
   reads when it composes the background.
2. **`loadImage('map_<mapArea>', '...')`** in `loadAssets`
   (`src/main.js`, the cluster around the existing `map_throne_room` /
   `map_grand_hall` lines). Without this preload, the image element
   never gets into `images[]` and the map view draws a blank frame
   even though `map.mapImages` is set.

Symptom of skipping step 2: the map background is broken but
encounter dialog backgrounds (which go through
`ENCOUNTER_BG_MAP` + the `bg_<name>` preload) still render fine on
the same art.

## Fog of war — the default for new maps

New maps use **standard fog of war** by default — do NOT add them to
`NO_FOG_MAPS` (that set is only for small/open layouts the player
should see whole at a glance, e.g. the Summit Ridge, a 2-node temple).

The standard pattern is the **`discoverable`** node flag (see
`createSouthOfQualibafMap` / `createRiverCaveMouthMap` /
`createQualibafBridgeMap` for live examples). Give every node past the
entry:

```js
{ ..., discoverable: true, hiddenName: '???', hiddenDescription: '…' }
```

Behavior, node by node:

1. **Invisible until one hop away.** A `discoverable` node isn't drawn
   at all until the party stands on one of its direct neighbors.
2. **`???` once one hop away.** It then draws as `???` (the
   `hiddenName`).
3. **Real name once walked on.** `arriveAtNode` clears a discoverable
   node's `hiddenName`/`hiddenDescription` on arrival (the
   `node.discoverable && !visitedNodes.has(...)` block), so visiting it
   reveals the real name. No extra wiring.

`discoverable` nodes are NOT locked — adjacency (the `connections`
graph) is what gates movement, so a linear chain of discoverable nodes
walks naturally with each one revealing as the party approaches. (The
older `isLocked` + `unlocks` style — used by dungeon maps like Filibaf
Forest — gates on *completing* the previous node instead of proximity;
prefer `discoverable` for overland/trail maps.)

**Revealing a name early (special case).** If a dialog names a node
outright (e.g. the North Crossroad says "Filibaf"), clear that node's
`hiddenName`/`hiddenDescription` in the post-encounter handler when the
dialog completes — an `unlocks`/proximity reveal would otherwise leave
it at `???` until the player physically walks onto it.

## Teleporter nodes — wire both directions by default

A "teleporter" is a map node with `passthroughTo` set. The standard
expectation is that it fires **both** on walk-onto **and** on
click-on-self, in **both** directions of the pair, **once unlocked**.
Don't gate either side on `isDone` or any "you walked here first"
heuristic — once the player can interact with the node at all, it
should teleport. The pair is symmetric: from A you reach B, and from
B you reach A, with no extra clicks.

When you add a new teleporter pair (or unlock a previously-locked
gate so it becomes a teleporter), the checklist is:

1. **Walk-onto** — add the matching cross-map branch in `arriveAtNode`
   (`src/main.js`, e.g. the `temple_moradin_door ↔ temple_moradin_entry`
   pattern at ~7886-7910). Both directions, fromNodeId-guarded against
   bounce-back.
2. **Click-on-self** — add both node ids to the `isCrossMapGate` ladder
   in `handleMapClick` (`src/main.js`, ~9150-9320). Gate on
   `!node.isLocked` (or whatever unlock latch you have), **not** on
   `node.isDone`. Same-map teleporters (`passthroughTo` to another
   node in the same `currentMap`) flow through the existing
   `hasActiveTeleport` check automatically — no entry needed.
3. **Map data** — set `passthroughTo` on both nodes pointing at each
   other, and put the paired id in each node's `connections` array so
   the visual line + adjacency check works.

If a teleporter ever feels asymmetric to the player ("I can walk
into it but not back out"), it's almost always because step 2 was
skipped — the cross-map gate runs only on walk-onto, never on the
self-click.

## Versioning

`GAME_VERSION` in `src/constants.js` is bumped manually before every push.

- Currently in the **2.x** range (Part 1 polish + post-launch fixes after
  the chapter-1 JS conversion finished at 2.0).
- Bump by **+0.01** per push (e.g. `2.01 → 2.02 → 2.03 → ... → 2.99`).
- Jump to **3.0** for the next major beat — Part 2 launch, or any other
  big content drop that justifies a clean version bump.
- Always bump before `git push`. The version appears on the title screen
  and in `trackEvent('game_start', { version })` analytics; stale
  versions in the wild make it hard to correlate user reports.

---
> Source: [EricPrieur/ccgQuest](https://github.com/EricPrieur/ccgQuest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
