## space-trader

> - Canvas 2D + `requestAnimationFrame`

# Space Trader Development Notes

## Stack

- Electron
- React + TypeScript
- Vite
- Canvas 2D + `requestAnimationFrame`

## Project Layout

### Electron Shell

- `electron/main.ts`
- `electron/preload.ts`

### React Entrypoints

- `src/main.tsx`
- `src/App.tsx`
- `src/App.css`
- `src/GameCanvas.tsx`

### Core Runtime

- `src/game/runtime.ts`
  - persistent runtime singleton
  - authoritative mutable game state
  - animation queue
  - pending targeting flow
  - local vs network command routing
  - worker-backed bot decisions
  - debug helpers
  - active content-selection + runtime-profile context
- `src/game/systems.ts`
  - `updateGame`
  - `renderGame`
- `src/game/types.ts`
  - frame, viewport, and animation types
- `src/game/derived.ts`
  - cached derived state keyed off runtime version
- `src/game/presentation.ts`
  - player/resource/faction presentation helpers

### Model Layer

- `src/game/model/state.ts`
  - canonical `GameState`
  - entity shapes
  - zones
  - initial-state creation
  - dynamic player-order bootstrap
  - elimination tracking
  - resource and runtime-profile aware defaults
  - deterministic generated-id counter
- `src/game/model/enums.ts`
  - phases, factions, resources, unit roles
  - note: factions/resources are dynamic string ids populated by loaded content
- `src/game/model/ids.ts`
  - typed IDs and player constants
- `src/game/model/hex.ts`
  - axial hex math
- `src/game/model/queries.ts`
  - state query helpers
- `src/game/model/selectors.ts`
  - UI selectors
- `src/game/model/migrations.ts`
  - state migration / hot-state repair
- `src/game/random/seeded.ts`
  - seeded RNG helpers for deterministic match setup

### Content System

- `src/game/content/loader.ts`
  - explicit set loading
  - registry reset / reload lifecycle
  - default built-in content initialization via `initializeDefaultContent()`
- `src/game/content/sets/catalog.ts`
  - built-in set manifest catalog
  - built-in set id lookup
  - default built-in selection (`alpha`, which depends on `foundation`)
- `src/game/content/registry.ts`
  - registered sets, cards, stack effects, factions, resources, maps, deck recipes, runtime profiles
- `src/game/content/cards/builders.ts`
  - shared card authoring helpers used across sets
- `src/game/content/sets/types.ts`
  - set manifests
  - faction/resource/runtime-profile module types
- `src/game/content/sets/foundation/**`
  - cardless shared gameplay foundation
  - shared stack effects, play effects, AI scoring, and runtime installers
- `src/game/content/sets/alpha/**`
  - first real playable set content and installers
  - depends on `foundation`

### Content Facades

- `src/game/content/cards/catalog.ts`
  - registry-backed card access
  - card metadata helpers
- `src/game/content/cards/types.ts`
  - generic card definition types
  - generic play-effect config / modifier shapes
- `src/game/content/stackEffects.ts`
  - registry-backed stack-effect access
- `src/game/content/stackEffects/types.ts`
  - generic stack behavior types
- `src/game/content/decks/starterDecks.ts`
  - starter deck access + validation
- `src/game/content/maps/catalog.ts`
  - registry-backed map access
- `src/game/content/mechanics/stateAccess.ts`
  - helper for namespaced mechanic state

### Mechanics

- `src/game/mechanics/index.ts`
  - generic mechanic-state lifecycle
- `src/game/content/sets/alpha/mechanics/**`
  - current live Alpha mechanics
  - `stealth`
  - `sprout`
  - `relay`
  - `surge`
  - `bloom`
  - `salvage`
  - `bastion`
  - `predation`
  - `emplaced`
  - `uncounterable`

### Registries

- `src/game/registries/triggerConditions.ts`
- `src/game/registries/autoTargets.ts`
- `src/game/registries/instructionHandlers.ts`
- `src/game/registries/playEffects.ts`
- `src/game/registries/cardPlayModifiers.ts`
- `src/game/registries/cardCounterability.ts`
- `src/game/registries/cardResolveAnimations.ts`
- `src/game/registries/directInteraction.ts`
- `src/game/registries/cascadeBranches.ts`
- `src/game/registries/combatHooks.ts`
- `src/game/registries/unitDeployment.ts`
- `src/game/registries/unitStatHooks.ts`
- `src/game/registries/mechanicInstructions.ts`
- `src/game/registries/mechanicAnimations.ts`
- `src/game/registries/mechanicState.ts`
- `src/game/registries/mechanicApis.ts`
- `src/game/registries/spellScoring.ts`
- `src/game/registries/stackEffectMagnitudes.ts`
- `src/game/registries/stackResolveAnimations.ts`
- `src/game/registries/stackPreviews.ts`
- `src/game/registries/boardBlastEffects.ts`
- `src/game/registries/debugStackResponses.ts`
- `src/game/registries/presentation.ts`

### Actions Pipeline

- `src/game/actions/commands.ts`
- `src/game/actions/events.ts`
- `src/game/actions/reducers.ts`
- `src/game/actions/instructions.ts`
- `src/game/actions/instructionHandlers.ts`
- `src/game/actions/handlers/cards.ts`
- `src/game/actions/handlers/combat.ts`
- `src/game/actions/handlers/phase.ts`
- `src/game/actions/handlers/selection.ts`

### Rules

- `src/game/rules/validators.ts`
  - command legality
- `src/game/rules/cardPlayOptions.ts`
  - shared legal-target enumeration
- `src/game/rules/cardPlayLegality.ts`
  - playability/stack targeting helpers

### Turn Management

- `src/game/turn/playerOrder.ts`
- `src/game/turn/phaseMachine.ts`
- `src/game/turn/stack.ts`
- `src/game/turn/autoFlow.ts`
- `src/game/turn/priorityStops.ts`

### Systems

- `src/game/systems/combat.ts`
- `src/game/systems/nodeControl.ts`
- `src/game/systems/harvesting.ts`
- `src/game/systems/victory.ts`
- `src/game/systems/keywords.ts`
- `src/game/systems/continuousEffects.ts`
- `src/game/systems/unitStats.ts`
- `src/game/systems/cascade.ts`
- `src/game/systems/replacementEngine.ts`
- `src/game/systems/triggerEngine.ts`

### AI

- `src/game/ai/minimaxBot.ts`
  - current default bot entry point
- `src/game/ai/minimax/**`
  - search, evaluation, generation, and simulation helpers
- `src/game/ai/minimaxBot.worker.ts`
  - background worker wrapper for live renderer bot decisions
- `src/game/ai/botDecisionWorkerProtocol.ts`
  - worker message contract
- `src/game/ai/mvpBot.ts`
  - legacy heuristic bot entry point
- `src/game/ai/mvpBot/**`
  - shared heuristic helpers and tactical/card-choice modules

### Networking

- `src/network/client.ts`
  - multiplayer session state
  - queue / command transport
  - SSE subscription and resync
- `src/network/protocol.ts`
  - shared client-side transport shapes
- `src/network/useMultiplayerSnapshot.ts`
  - React subscription hook

### Server

- `server/src/index.ts`
  - Node server entry point
- `server/src/matchmaker.ts`
  - format-aware queues and match creation
- `server/src/matchRoom.ts`
  - authoritative room state, player-order fanout, and command handling
- `server/src/createMatchState.ts`
  - seeded initial match creation
- `server/src/sessionStore.ts`
  - reconnectable player sessions
- `server/src/roomStore.ts`
  - live room lookup
- `server/src/protocol.ts`
  - server transport types

### Render

- `src/game/render/layout.ts`
- `src/game/render/primitives.ts`
- `src/game/render/grid.ts`
- `src/game/render/entities.ts`
- `src/game/render/overlays.ts`
- `src/game/render/animations.ts`
- `src/game/render/animationDrawing.ts`

### UI

- `src/ui/GameHudPanels.tsx`
- `src/ui/GameTopBar.tsx`
- `src/ui/HandTray.tsx`
- `src/ui/CommandStackPanel.tsx`
- `src/ui/ResourceIcon.tsx`
- `src/ui/useGameSnapshot.ts`

### Docs

- `game-design.md`
- `architecture.md`
- `instructions.md`
- `networked-multiplayer-feature.md`
- `four-player-refactor.md`
- `three-player-feature.md`
- `attack-refactor.md`
- `layers-refactor.md`
- `faction-identity.md`
- `new-cards.md`
- `faction-refactor.md`

## Commands

- `npm run dev`
- `npm run build`
- `npm run preview`
- `npm run typecheck`
- `npm test`
- `npm run test:watch`
- `npm run server:build`
- `npm run server:start`

### Introduction Screenshots

To regenerate the annotated tutorial screenshots in `docs/introduction/`:

1. Start the dev server in one terminal: `npm run dev`
2. In another terminal: `npx tsx scripts/capture-introduction-screenshots.ts`

The script launches a Playwright Chromium browser, injects specific game states for each tutorial step, draws SVG arrow annotations pointing to key UI elements, and saves screenshots to `docs/introduction/`. The game runtime is exposed on `window.__gameRuntime` in dev mode. If the UI layout, hex grid rendering, or HUD components change, re-run the script and verify the 12 output images.

## Current Architecture Decisions

- Canonical gameplay state lives in `src/game/runtime.ts`, not React state.
- Gameplay mutations flow through:
  - commands
  - events
  - instructions
  - triggers
- Live phase loop:
  - `start`
  - `economy`
  - `main`
  - `tactical`
  - `end`
  - `discard`
- Current live economy defaults:
  - the starting player starts with `2 currency + 2 primary`
  - non-starting players start with `5 currency + 2 primary`
  - deposits are `2`
  - passive economy is `+1 currency`
- Player identity is runtime-seat based:
  - `state.playerOrder` defines turn / priority seating
  - `state.eliminatedPlayerIds` removes players from live rotation
  - 1v1 remains `player_1`, `player_2`
  - 3-player FFA uses `player_1` through `player_3`
  - 4-player FFA uses `player_1` through `player_4`
- Continuous effects are layered and authoritative for stat/keyword changes.
- Mechanic-owned state lives under `state.mechanicState` in:
  - `match`
  - `turn`
  - `resolution`
- Content is explicitly loaded through `content/loader.ts`.
- Built-in set manifests are selected through `content/sets/catalog.ts`.
- Default built-in content is `alpha`, which pulls in the cardless `foundation` dependency.
- Shared reusable gameplay primitives live in `foundation`; cards, decks, maps, and runtime profiles live in real sets such as `alpha`.
- Runtime can now be created or reset from explicit content bundles through:
  - `loadConfiguredContentSets(...)`
  - `createConfiguredRuntime(...)`
  - `GameRuntime.resetWithContent(...)`
- Runtime defaults come from registered runtime profiles, not kernel constants.
- Networked multiplayer is server-authoritative command replay, not client-authoritative state sync.
- Online matchmaking is format-aware:
  - `pvp_1v1` uses `alpha_default`
  - `ffa_3p` uses `alpha_three_player`
  - `ffa_4p` uses `alpha_four_player`
- Current live Alpha runtime profiles:
  - `alpha_default` on `frontier_belt`
  - `alpha_three_player` on `frontier_triad`
  - `alpha_four_player` on `frontier_crossroads`
- Resource modules now own:
  - `kind` such as currency vs primary
  - display order
  - glyph data
  - theme data
- Card resolution is data-driven through:
  - card play metadata in `catalog.ts`
  - generic stack behavior definitions
  - generic play-effect registries
  - set-owned installers for AI/animation/preview/debug behaviors
- `StackResolutionRules` is not the mental model anymore.
- Generated ids that affect sync must come from stable sources:
  - stable card-instance ids where possible
  - otherwise `state.nextGeneratedIdCounter`
- Never derive authoritative ids from mutable state like `log.length`.
- Current state version is `26`.

## Current Faction Mechanics Snapshot

- Alloy:
  - `bastion`
  - `salvage`
  - `emplaced`
  - formation / siege / damaged-matters shell
- Flux:
  - `relay`
  - `surge`
  - stack / spellchain / spatial combo shell
- Biomass:
  - `sprout`
  - `bloom`
  - `predation`
  - swarm / growth / board-to-resource shell

## `getGameRuntime` Contract

- `getGameRuntime()` returns the same runtime instance for the life of the renderer session.
- React should subscribe to runtime snapshots, not copy gameplay state into component state.
- `GameCanvas` owns the RAF loop and calls runtime step/render plumbing.
- New authoritative gameplay state belongs in `state.ts` plus `migrations.ts`.

## HMR Workflow

- Runtime instance persists through HMR.
- Simulation/render logic can be hot-swapped without wiping the match.
- State schema changes should always be accompanied by migration updates.
- Registry-backed content can be reset and reloaded deterministically.
- Server/client multiplayer bugs are often determinism bugs; check ids, seeds, and content parity before assuming transport failure.
- Online hidden information is still trust-based: clients can reconstruct hidden zones from deterministic setup and command replay. Secure hidden-state views are a separate future project.

## Current Extension Points

- new content sets through `CardSet` manifests
- new runtime installers through `installers`
- new mechanics through set-owned mechanic modules
- new resources/factions/maps/decks/runtime profiles through the content registry
- new effect families through generic play-effect / stack-effect registrations
- new AI/animation/preview behaviors through registries instead of kernel switches

## Architectural Pressure Points

These still imply meaningful engine work, not just content:

- graveyard / reanimation / recursion UX and rules
- true token support
- multi-target choice cards
- explicit support for deterministic infinite combos
- secure online hidden-information networking
- full content-context isolation beyond the current process-global registries
- any major change to spell-damage-vs-armor rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/williamcotton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
