## rtchess

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RTChess is a real-time multiplayer chess game built in Unreal Engine 5.7 (C++). Pieces move and attack in real time with an energy system gating actions. The architecture is server-authoritative with client-side prediction.

## Building

This is a standard UE5 project — there are no custom build scripts. Use one of:

- **Visual Studio:** Open `RTChess.sln`, build from IDE (Development Editor / Win64)
- **UnrealBuildTool:** `UnrealBuildTool.exe RTChessEditor Win64 Development "RTChess.uproject"`
- **Unreal Editor:** File → Generate Visual Studio Project Files, then build via Editor or VS

Module rules: `Source/RTChess/RTChess.Build.cs`
Key dependencies: `Core, CoreUObject, Engine, InputCore, NavigationSystem, AIModule, Niagara, EnhancedInput, UMG`

## Source Layout

```
Source/RTChess/
├── Core/          # GameMode, GameState, GameInstance, PlayerController, PlayerState, Subsystem
├── Board/         # ChessBoard (8x8 grid, instanced mesh), BoardRulesComponent (move validation)
├── Pieces/        # ChessPieceBase (APawn) + 6 concrete piece types
├── Components/    # Movement, animation, selection, texturing, tile-indicator components
├── Animation/     # AnimInstance classes and AnimNotifies
├── Data/          # CommonDataHeader (all enums/structs), DataAssets for pieces/board/VFX
├── UI/            # MainHUDWidget, EnergyBarWidget, SideUI (3D world-space UI)
├── Tile/          # TileIndicator + TileIndicatorManagerComponent (pooled, 64 default)
└── Player/        # PlayerPawn, RTChessPlayerController (also lives in Core/)
```

## Architecture

### Game Flow
`ARTChessGameMode` spawns and positions all pieces at match start using `BoardSetupDataAsset`. `ARTChessGameState` holds replicated match-end info (`FMatchEndInfo`) and broadcasts delegates on state changes.

### Chess Pieces
All pieces inherit from `AChessPieceBase` (APawn). Each piece owns:
- `PieceMovementComponentBase` (or `PieceJumpMovementComponent` for Knight) — handles linear/jump movement
- `PieceAnimationControlComponent` / `KnightAnimationControlComponent` — drives `EPieceAnimState`
- `SelectableComponent` — stencil highlight (0=friendly, 1=selected, 2=enemy)
- `PieceTextueringComponent` — material assignment per team
- `TileIndicatorManagerComponent` — shows/hides legal move tiles

Pure virtual methods `GetValidMoveTargets()` and `GetValidAttackTargets()` are implemented per piece type.

### Energy System
Defined in `ARTChessPlayerState`: max 4.0, regen 0.25/tick, cost 1.0 per action. Uses client-side prediction — the client deducts energy immediately (with a sequence number `ConfirmedConsumeSeq`) and reconciles against server confirmation via `FEnergySync`.

### Networking
- `CommandRPC` (Server RPC): `ARTChessPlayerController` → server for move/attack commands
- `Multicast_PlayHitEffect`: broadcast hit Niagara effects to all clients
- Replicated: team assignment, energy, piece positions, match state
- Command flow: client clicks tile → `PlayerController` builds `FCommandData` → validates energy → `CommandRPC` → server validates via `BoardRulesComponent` → applies movement/attack

### Data-Driven Configuration
- `PieceDefinitionDataAsset` — maps `EPieceType + EPieceTeam` → Blueprint class
- `BoardSetupDataAsset` — initial piece layout per team (array of `FBoardPieceSetupData`)
- `ChessGameSetupDataAsset` — top-level game setup
- `RTChessVFXSettingsDA` — Niagara system references
- `PieceDefinitionSubsystem` — world subsystem that loads and caches these assets

### Key Data Types (`Data/CommonDataHeader.h`)
```cpp
ECommandType   { Move, Attack, Invalid }
EPieceType     { King, Queen, Rook, Bishop, Knight, Pawn }
EPieceTeam     { White, Black, Invalid }
EPieceAnimState{ Idle, Moving, Attacking, Dead }
EJumpPhase     { None, Ready, Up, Down, Recover }   // Knight only
FTileCoord     { int32 Row, Col }
FCommandData   { ECommandType Type, FTileCoord Destination }
FMatchEndInfo  { bool bEnded, EPieceTeam WinnerTeam, FString Reason }
```

### Board Coordinate System
`AChessBoard` manages an 8x8 grid. World position = `FTileCoord × CellStep (150.f)` + board origin. Board plate height is `150.f`. Tile occupancy is tracked per cell; `BoardRulesComponent` reads occupancy to validate legal moves.

### Knight Jump Movement
`PieceJumpMovementComponent` phases: `Ready` (1s pause) → `Up` (arc up) → `Down` (arc down) → `Recover`. `JumpReadyAnimNotify` signals the anim blueprint to transition phases.

### Input
Uses Enhanced Input System. Key actions in `ARTChessPlayerController`:
- Camera movement (axis)
- Destination click (move selection)
- Target click (attack selection)
- Edge scrolling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jidungg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jidungg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
