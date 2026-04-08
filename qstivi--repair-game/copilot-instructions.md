## repair-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Repair Game** is a settlement/base-building strategy game built with Unity 6000.0.28f1. Players manage resources, construct and repair buildings, and defend their main structure through a production chain system with card-based upgrades.

## Build & Run

Open in Unity Hub or Unity Editor (version 6000.0.28f1 or compatible). No external build commands needed - use Unity's standard build pipeline.

**Scenes:**
- `Assets/Scenes/Weltkarte.unity` - World map (main menu/level select)
- `Assets/Scenes/Level/Level 1.unity`, `Level 2.unity` - Playable levels

## Architecture

### Core Systems

**Singleton Managers:**
- `GameVariables.Instance` - Global game state
- `LevelDataMananger.Instance` - Level data access (note: intentional typo preserved)

**Event System (`EventManager.cs`):**
- `SyncTickEverySecond` - Timer synchronization
- `UpdateStorage` / `UpdateRessources` - Resource change notifications

### Building System

Template method pattern with inheritance hierarchy:
```
AbstractBuilding
├── AbstractMainBuilding
├── AbstractProductionBuilding
│   ├── AbstractHouse (residential)
│   ├── AbstractStoneMason (stone production)
│   └── AbstractWoodcutter (wood production)
└── AbstractDefenseBuilding
    └── AbstractWall
```

Each building type has Level1, Level2, Level3 concrete implementations in `Assets/Scripts/Bulidings/` (note: directory typo is intentional in codebase).

### Stat System (`Stat.cs`, `Modifier.cs`)

Generic `Stat<T>` with modifier support:
- Absolute modifiers
- Relative additive modifiers
- Relative multiplicative modifiers
- Dirty flag caching with `OnStatChanged` events

### Resource System (`Cost.cs`)

Four resource types: Coins, Wood, Stone, PotionTools. Implements arithmetic operators (+, -, /) and comparison (>=, <=).

### Card System (`Assets/Scripts/Cards/`)

Polymorphic cards: EffectCard, EquipmentCard, IngredientCard, UnitCard. Uses behavior pattern (`ICardBehavior`) for unlock/upgrade conditions.

### Level Configuration

- `LevelConfig` (ScriptableObject) - Immutable level definition
- `LevelState` - Mutable runtime state (current resources, etc.)
- `BuildingSpot` - Placeable building locations with unlock costs

### Damage Types (`Assets/Scripts/Enums/DmgType/`)

14 damage types in categories:
- Physical: Bludgeoning, Piercing, Slashing
- Elemental: Acid, Cold, Fire, Lightning, Poison, Thunder, Water
- Magical: Force, Necrotic, Psychic, Radiant

## Testing

NUnit tests in `Assets/Tests/`. Run via Unity Test Runner (Window > General > Test Runner).

## Key Interfaces

- `IStat<T>` - Stat system interface
- `ICardBehavior` - Card behavior pattern
- `IUpgradable` - Building upgrade interface
- `IStorage` - Storage system interface

## Code Conventions

- Mixed German/English naming (German comments, some German variable names)
- SerializeField for Unity inspector bindings
- Async/await for long-running operations (TimerManager)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qStivi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qStivi)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
