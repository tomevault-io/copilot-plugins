## goblins-eat-the-king

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Goblins Eat the King** is a cooperative multiplayer top-down dungeon crawler built with Unity (2D) and **Netcode for GameObjects (NGO)**. Players explore procedurally generated dungeons, fight enemies, collect coins, and upgrade stats between runs.

## Unity Development

This is a Unity project — there are no build/test CLI commands. All development happens through the Unity Editor:

- Open the project in Unity (check `ProjectSettings/ProjectVersion.txt` for the required Unity version)
- Play mode testing is done directly in the Unity Editor
- Builds are created via **File → Build Settings** in the Editor
- The multiplayer simulation tool (Unity Multiplayer Play Mode package) allows testing with multiple clients in the editor simultaneously

## Architecture

### Network Authority Model

All game state is **server-authoritative**. This is the single most important architectural constraint:
- Damage, stat changes, and enemy AI run on the server only
- Clients send `ServerRpc` requests; server validates and applies changes
- Results are broadcast to clients via `ClientRpc` or `NetworkVariable` callbacks
- Never apply gameplay state changes directly on the client

### Core Systems

**PlayerController** (`Assets/Scripts/PlayerController.cs`) is the central stat hub — it holds ~20 `NetworkVariable` stats (HP, MP, Endurance, Defense, CritRate, etc.), handles incoming `HitData`, applies damage/effects, and manages regeneration. All damage flows through `ApplyHit(HitData)` on the server.

**Combat System** (`Assets/Scripts/Combat/`) is fully data-driven:
- `AttackDefinition` (ScriptableObject) defines an attack's damage, range, timing, effects, and weight
- `EnemyAttackBrain` selects attacks at runtime by filtering eligible ones (range + cooldown), then picks weighted-randomly
- `IEnemyAttack` interface — implemented by `ConeAttack`, `AuraPulseAttack`, `RandomSpikesAttack`, `ProjectileAttack`
- `HitData` struct carries damage + knockback + a list of `AttackEffect` (Slow, Stun, Bleed, Poison)
- To add a new attack type: implement `IEnemyAttack`, create an `AttackDefinition` asset, assign to enemy in Inspector

**Dungeon Generation** (`Assets/Scripts/Dungeon/`):
- `DungeonGenerator` uses a Growing Tree algorithm seeded for determinism — all clients generate the same map from the same seed
- `RoomBuilder` + `RoomFurnisher` place tiles, doors, and decorations deterministically
- Enemies scale per room (hp/damage/speed multipliers); `DungeonGenerator.NotifyEnemyDied()` tracks room clear state
- On full clear, a `RewardBubble` spawns → player collects it → `UpgradeChoice` UI appears

**Progression:**
- `StatUpgradeManager` (persistent between runs via `PlayerPrefs`) — 5 upgrade levels per stat, cost curve: 10→25→50→100→200 coins
- `PowerupDatabase` (ScriptableObject) — in-dungeon upgrade/downgrade definitions shown by `UpgradeChoice`
- `CoinManager` singleton tracks session and total coins

**Singletons:** `CoinManager`, `StatUpgradeManager`, `SoundManager`, `GameUI`, `DungeonGenerator` — all use `Instance` pattern. Avoid creating duplicate instances.

### Adding a New Enemy Attack Type

1. Create a new C# class implementing `IEnemyAttack`
2. Create an `AttackDefinition` asset (right-click → Create → Combat → Attack Definition)
3. Set `AttackType` to match and fill parameters in the Inspector
4. Add the script component to the enemy prefab alongside `EnemyAttackBrain`
5. Add the `AttackDefinition` asset to the enemy's `attackDefinitions` list in the Inspector

### Networked Object Rules

- Any prefab spawned at runtime via `NetworkObject.Spawn()` must be registered in the **NetworkPrefabsList**
- Set initial state (velocity, damage, lifetime) **before** calling `Spawn()` so it's included in the spawn message — this avoids jitter on projectiles
- Only the server should call `Spawn()` / `Despawn()`

### Key Script Locations

| System | Script(s) |
|---|---|
| Player stats & damage | `Assets/Scripts/PlayerController.cs` |
| Player movement & knockback | `Assets/Scripts/PlayerMovement.cs` |
| Enemy AI & state machine | `Assets/Scripts/EnemyController.cs` |
| Attack selection | `Assets/Scripts/Combat/EnemyAttackBrain.cs` |
| Attack data | `Assets/Scripts/Combat/AttackDefinition.cs` |
| Hit/effect data | `Assets/Scripts/Combat/HitData.cs` |
| Dungeon generation | `Assets/Scripts/Dungeon/DungeonGenerator.cs` |
| Room building | `Assets/Scripts/Dungeon/RoomBuilder.cs`, `RoomFurnisher.cs` |
| Stat upgrades (persistent) | `Assets/Scripts/StatUpgradeManager.cs` |
| In-dungeon upgrades | `Assets/Scripts/Interactables/UpgradeChoice.cs` |
| Coin system | `Assets/Scripts/CoinManager.cs` |
| HUD | `Assets/Scripts/UI/GameUI.cs` |
| Sound | `Assets/Scripts/SoundManager.cs` |
| Sprite depth sorting | `Assets/Scripts/SortByY.cs` |

### Depth Sorting

Top-down depth is handled by `SortByY.cs` which updates each object's `sortingOrder` each frame based on Y position. Attach this to any sprite that needs proper overlap behavior (players, enemies, decorations).

### Localization

`LocalizationManager` handles French/English text for stat names, effect descriptions, and UI labels. When adding new UI text, route it through the localization system rather than hardcoding strings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fmaret) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
