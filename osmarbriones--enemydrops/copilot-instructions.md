## enemydrops

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EnemyDrops is a BepInEx 5.x mod for the Unity game **R.E.P.O.** that spawns loot items when enemies die. It uses HarmonyLib 2.4.1 for non-invasive patching of game code and Photon PUN 2 for multiplayer item spawning. Only the host runs the drop logic; clients see synced results automatically.

## Build and Deploy

```bash
# Debug build (auto-deploys to BepInEx plugins folder via PostBuild)
dotnet build EnemyDrops.csproj

# Release build
dotnet build -c Release EnemyDrops.csproj
```

The `PostBuild` target copies `EnemyDrops.dll` to the Steam BepInEx plugins folder and the r2modman Debug profile plugins folder.

Testing is done in-game — build, launch R.E.P.O., and observe behavior. There is no automated test suite.

## Game Reference Files

Two environment variables point to local read-only reference material (not in the repo):

| Variable | Contents |
|---|---|
| `REPO_DECOMPILED` | ILSpy-exported C# source from `Assembly-CSharp.dll` |
| `REPO_ASSETS` | AssetRipper-extracted game assets |

Use these when you need to understand game internals (item pickup logic, `StatsManager`, `SemiFunc`, etc.) that aren't visible from the mod code alone.

## Dependencies

All references are local paths to game assemblies and Harmony. Ensure these exist before building:
- `Assembly-CSharp.dll` and related UnityEngine DLLs — from the R.E.P.O. game install
- `0Harmony.dll` (v2.4.1) — from a relative path defined in the `.csproj`
- `BepInEx.dll` — from the game's BepInEx install

## Architecture

The full data flow is documented in `ARCHITECTURE.md`. Here is the cross-file picture:

### Runtime Flow

```
Level Start
  └─ ReloadDropTablesOnLevelStart (patch on EnemyDirector.Start)
       ├─ ConfigurationController.Reload()      ← reads .cfg file
       ├─ ItemDropper.ResetForNewLevel()         ← resets s_dropsThisLevel counter
       └─ DroppedInstanceTracker.ClearForNewLevel()

Enemy Death
  └─ EnemyDeathPatch (patch on EnemyHealth.Awake → subscribes to onDeath)
       └─ ItemDropper.TrySpawnForEnemy()
            ├─ checks s_dropsThisLevel < MaxDropsPerLevel
            ├─ EnemyDifficultyAccessor.GetDangerLevel()  ← reflection into internal field
            ├─ IsExcludedEnemy()                          ← skips Gnome, Banger, etc.
            ├─ ItemDropTables.GetWeightsFor(difficulty)  ← weighted table per difficulty
            ├─ ItemProvider.TrySpawnByKey()               ← Photon-synced instantiation
            └─ DroppedInstanceTracker.MarkDropped()       ← adds DroppedItemTag component

Item Name Assigned (Photon sync)
  └─ PunManager_SetItemNameLOGIC_Patch
       └─ DroppedInstanceTracker.RegisterInstance(name)  ← links GameObject to battery key

Scene Switch
  └─ ClearDroppedBatteriesOnSceneSwitch (patch on SemiFunc.OnSceneSwitch)
       └─ DroppedInstanceTracker.ClearBatteriesForDroppedInstances()
            ├─ removes/resets tracked entries in StatsManager.itemStatBattery
            └─ ClearForNewLevel()
```

### Key Design Decisions

- **`ConditionalWeakTable` in `EnemyDeathPatch`**: Prevents subscribing to `onDeath` multiple times per `EnemyHealth` instance across reloads.
- **`DroppedItemTag` marker component**: Used as a runtime tag on GameObjects; the only link between a spawned item and "this came from EnemyDrops."
- **Instance name registration is deferred**: The battery dictionary (`StatsManager.itemStatBattery`) is keyed by instance names like `"Item Gun Shotgun/1"` which are assigned asynchronously by `PunManager.SetItemNameLOGIC`, not at spawn time.
- **Host-only guard**: Every drop decision is gated on `SemiFunc.IsMasterClientOrSingleplayer()`.

### Item Keys

All valid item key strings are defined as constants in `Providers/ItemKeys.cs`. Use `ItemKeys.<Name>` rather than raw strings anywhere items are referenced. `ItemKeys.All` is the flat array used to build config entries.

### Configuration

Generated at `BepInEx/config/osmarbriones.EnemyDrops.cfg`. Contains `MaxDropsPerLevel` and per-difficulty weighted item tables. Config is reloaded each level start, not just once at plugin load.

---
> Source: [OsmarBriones/EnemyDrops](https://github.com/OsmarBriones/EnemyDrops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
