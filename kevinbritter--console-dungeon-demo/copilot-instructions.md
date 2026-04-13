## console-dungeon-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Console Dungeon is a minimal, extendable console-based roguelike demo written in C# targeting .NET 8. The project emphasizes clean code architecture, easy extensibility, and a polished console presentation through a text-based UI.

## Build & Run Commands

**Run the game:**
```bash
dotnet run --project Console_Dungeon
```

**Run all tests:**
```bash
dotnet test
```

**Build the solution:**
```bash
dotnet build
```

**Run from Visual Studio:**
Press F5 to build and run.

## Core Architecture

### Control Flow Model

The game uses a **MenuAction-based navigation system** instead of traditional boolean returns:

1. **Program.cs** contains the main application loop that switches on `MenuAction` enum values
2. All menus implement `IMenu` interface with a `Show()` method that returns a `MenuAction`
3. `MenuAction` values (Main, Play, Options, Exit, NewGame, etc.) drive all navigation
4. This keeps control flow explicit, testable, and easy to extend

**Flow:** Main Menu → Play Menu → Character Creation → GameLoop → (on death/exit) → Main Menu

### GameState & Serialization

`GameState` is the central data container designed for serialization from day one:
- Contains `Player`, `DungeonLevel`, `Seed`, timestamps, turn count, and flags dictionary
- Marked `[Serializable]` for future save/load implementation (Phase 6)
- Instantiated fresh for new games, will be loaded from JSON for continue games
- Passed by reference to handlers and loops rather than owned by menus

### GameLoop Architecture

`GameLoop.Run()` is the core turn-based game loop:
1. Render current game state
2. Get player input via `InputHandler`
3. Resolve action (movement, combat, rest, status, exit)
4. Update world state
5. Check win/death conditions
6. Repeat

The loop delegates to specialized handlers:
- **MovementHandler** - handles player movement and room transitions
- **EncounterHandler** - triggers and manages combat/encounters
- **CombatEngine** - resolves combat calculations
- **LootManager** - handles item drops and rewards

### Manager Pattern

Data is loaded from JSON files in the `Data/` folder via static manager classes:
- **CharacterClassManager** - loads `CharacterClasses.json`
- **EncounterManager** - loads `Encounters.json`
- **MessageManager** - loads `Messages.json` (supports placeholder replacement like `{name}`, `{level}`)
- **RoomDescriptionManager** - loads `RoomDescriptions.json`
- **ItemManager** - loads `Items.json`
- **MetaUnlockManager** - manages persistent unlocks across runs

All managers are loaded once in `GameLoop` constructor before the game loop starts.

### Action Pattern

Player actions are implemented as classes implementing `IGameAction`:
- **RestAction** - player rests to recover health
- **ShowStatusAction** - displays player stats and inventory
- **ShowLevelUpAction** - displays level-up celebration screen

This pattern keeps game logic modular and testable.

### Level Generation

`DungeonLevel` generates procedural dungeon layouts:
- Uses a random walk algorithm starting from the center
- Creates connected paths through a 5x5 grid (configurable)
- Places boss rooms at maximum distance from start
- Tracks visited/unvisited rooms for fog of war
- Boss room only visible on map when within one room distance
- Deeper levels (higher `LevelNumber`) increase size from 5x5 to 5x10 grids for longer runs

### UI Rendering

**ScreenRenderer** is the core UI system:
- Draws fixed-size bordered screens (asterisk borders)
- `ScreenRenderer.BoxWidth` and `BoxHeight` define the rendering area
- `DrawScreen(string body)` renders content with default header/footer
- `RenderMap(DungeonLevel, Player)` generates ASCII dungeon maps
- Output is captured via `ScreenRenderer.Output` (TextWriter) for testing
- `Console.Clear()` is guarded to prevent test failures

**TextFormatter** handles text wrapping to fit screen width.

Console size is enforced on startup in `Program.cs:EnsureConsoleSize()` to ensure the game box fits properly.

## Data-Driven Design

Game content lives in `Console_Dungeon/Data/`:
- **CharacterClasses.json** - class definitions with starting stats
- **Encounters.json** - enemy types, combat scenarios, descriptions
- **Items.json** - weapons, armor, consumables
- **Messages.json** - UI text with placeholder support (e.g., `{name}`, `{gold}`)
- **RoomDescriptions.json** - room flavor text

JSON files are copied to output directory via `.csproj` configuration:
```xml
<None Update="Data\**\*.json">
  <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</None>
```

## Testing Approach

Test project: `Test_Console_Dungeon` using xUnit framework.

**Key testing patterns:**
- UI tests capture output via `ScreenRenderer.Output` instead of redirecting `Console.Out`
- Instance menus (not static) allow dependency injection and mocking
- Keep rendering logic stateless and pure for easier validation
- Tests verify layout, text wrapping, and formatting correctness

## Development Phases (Roadmap)

The project follows a phased implementation plan documented in `ROADMAP.md`:

- **Phase 1** (✓ Implemented 01/25/26): Foundations - GameState, menu flow, game loop
- **Phase 2** (✓ Implemented 01/26/26): Minimal dungeon & movement system
- **Phase 3** (✓ Implemented 01/27/26): Player identity - stats, classes, character creation
- **Phase 4** (✓ Implemented 01/30/26): Combat system with turn-based encounters
- **Phase 5** (✓ Implemented 01/31/26): Inventory & items - equipment, loot drops
- **Phase 6** (Planned): Persistence - save/load system via JSON serialization
- **Later Phases**: Meta progression, abilities, achievements

## Important Architecture Notes

### Menu System
- Menus are instance classes (not static) to support dependency injection
- Menus return `MenuAction` to indicate navigation outcome
- `Program.cs` main loop handles menu transitions based on returned actions

### Player Progression
- XP system with level-ups displayed via `ShowLevelUpAction`
- Level-up info includes stat gains and new level
- `Player.HasPendingLevelUp` checked after encounters and movement
- Progress bar rendered with `Player.GetXPProgressBar(width)`

### Combat System
- Combat runs as a sub-loop within `EncounterHandler`
- `CombatEngine` handles damage calculations
- Player can attack, flee, or inspect during combat
- Fleeing returns player to previous room to retry

### Level Progression
- Boss rooms only spawn when all other rooms are explored
- Defeating boss triggers level completion screen
- Player advances to next dungeon level with 25% health restoration
- New level uses `Seed + LevelNumber` for deterministic generation

### Known Issues & TODOs
- Continue game functionality is stubbed (disabled until Phase 6)

## Testing Preferences

**Important:** The user prefers to run tests themselves via the IDE.
- When code changes are made, remind the user to run tests
- DO NOT offer to run tests via `dotnet test` commands
- The user will use Visual Studio Test Explorer or run tests manually

## Code Style Preferences

- C# 12 with .NET 8.0 target
- Nullable reference types enabled
- Prefer instance classes over static for testability
- Keep UI rendering stateless
- Follow existing naming conventions in `UI/` and `Menus/`
- Single-responsibility commits preferred

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KevinBRitter)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KevinBRitter)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
