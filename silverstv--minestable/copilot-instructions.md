## minestable

> This repository contains a Unity game project.

# AGENTS.md

## Project overview

This repository contains a Unity game project.
Primary goal: make safe, minimal, testable changes that preserve project stability and avoid unnecessary editor/asset churn.

## Source of truth

Treat the existing codebase and project structure as the source of truth.
Prefer adapting to the current architecture over introducing a new one.
Do not rewrite broad subsystems unless explicitly asked.

## Repository areas

Main working areas:

- `Assets/` — gameplay code, MonoBehaviours, ScriptableObjects, editor scripts, prefabs, scenes, art-related assets
- `Packages/` — package manifests and package settings
- `ProjectSettings/` — Unity project settings
- `.agents/skills/` — repo-scoped Codex skills
- `.codex/` — project-scoped Codex configuration

## Files and folders to avoid

Do not modify these unless the user explicitly asks:

- `Library/`
- `Temp/`
- `Logs/`
- `Obj/`
- `Build/`
- `Builds/`
- generated caches
- auto-generated IDE files

Do not delete `.meta` files.
Do not rename or move assets unless explicitly requested.

## Safety rules for Unity changes

Prefer changes in this order:

1. C# scripts in `Assets/**/*.cs`
2. asmdef files when needed
3. `Packages/manifest.json` only when required
4. `ProjectSettings/*` only when required
5. scenes, prefabs, ScriptableObject assets only on explicit request

When touching serialized Unity files (`.unity`, `.prefab`, `.asset`, `.anim`, `.controller`):
- keep changes minimal
- avoid broad formatting churn
- do not mass-rewrite unrelated serialized fields

## Coding conventions

- Use clear, small, focused classes and methods
- Preserve existing naming/style unless explicitly asked to standardize
- Avoid overengineering
- Prefer composition over deep inheritance
- Avoid hidden side effects
- Keep runtime code and editor-only code separated
- Put editor-only code under `Editor/` folders when relevant
- Avoid allocations in hot paths where possible
- Avoid `FindObjectOfType` / `FindObjectsOfType` in frequently executed gameplay code unless justified
- Avoid repeated `GetComponent` calls inside update loops unless cached
- Avoid LINQ in hot gameplay loops unless performance impact is irrelevant

## Architecture preferences

Unless asked otherwise:

- Keep gameplay logic decoupled from view code
- Do not move core logic into MonoBehaviours if it can stay in plain C# classes
- MonoBehaviours should mainly compose scene references, lifecycle hooks, and integration points
- ScriptableObjects are acceptable for static data/config
- Prefer deterministic logic when the system already relies on seeds or reproducible generation

## How to work on tasks

For each task:

1. Read the relevant files first
2. Identify the smallest safe change
3. Implement only what is needed
4. Preserve backward compatibility unless asked otherwise
5. Summarize what changed and what still may need verification

## Before editing

Before large edits:
- inspect related scripts
- check for asmdef boundaries
- check for namespace consistency
- check whether the target code is runtime, editor, or test code

## Testing and verification

When possible, verify with the least invasive method available.

Preferred checks:
- compile-level consistency
- obvious namespace/reference correctness
- Unity-specific file placement correctness
- note any steps that should be verified in the Unity Editor

If you cannot run Unity or editor-specific validation, say so explicitly.
Do not claim the scene/prefab/editor behavior was verified unless it actually was.

## Git hygiene

Prefer small diffs.
Do not modify unrelated files.
If a change is risky, say so clearly.
If a file likely has high merge-conflict risk (scene/prefab/package manifest), mention it in the summary.

## Response expectations

When finishing a task, report:
- changed files
- what was changed
- possible risks
- what should be checked in Unity Editor

## Project mental model

This is a 2D side-view mining game with a grid-based world.

Core systems:
- MineGrid: pure C# grid model of the mine
- CellState: value struct for a single cell
- BlockDatabase: ScriptableObject with block configs
- MineGenerator: creates initial mine content from seed
- CaveGenerator: generates empty spaces
- VeinGenerator: generates ore veins
- CollapseSystem: handles cave-ins and support checks
- MiningController: applies player mining actions
- InventoryService: stores collected resources
- MineView: renders grid cells to scene objects / tilemaps
- GameManager: connects scene objects to runtime services
- ItemId: 

Rules:
- Keep gameplay rules in plain C# services when possible
- MonoBehaviours should mainly connect Unity scene references to gameplay services
- Do not move deterministic generation logic into MonoBehaviours

## Directory map

- `Assets/Game/Scripts/Gameplay/` — pure gameplay logic, no MonoBehaviour dependencies
- `Assets/Game/Scripts/Root/` — gameplay orchestration
- `Assets/Game/Scripts/View/` — scene-facing MonoBehaviours and rendering glue
- `Assets/Game/Data/Blocks/` — ScriptableObjects with block definitions
- `Assets/Game/Art/Blocks/` — block sprites
- `Assets/Game/Art/UI/` — UI sprites
- `Assets/Game/Prefabs/World/` — world prefabs
- `Assets/Game/Prefabs/UI/` — UI prefabs
- `Assets/Game/Scenes/` — scenes

## Core game concepts

Important domain enums:

- `BlockType` describes the type of block stored in a world cell.
  Examples: Air, Dirt, Stone, Ore, etc.
  It answers the question: "What block is currently placed in this cell?"

- `itemID` describes the type of item that can exist in inventory, be dropped, collected, or used by gameplay systems.
  It answers the question: "What item does the player receive or carry?"

## Relationship between BlockType and itemID

`BlockType` and `itemID` are not the same concept.

- `BlockType` is world-state data.
- `itemID` is inventory/drop data.

A block may produce an item when mined, but block type and dropped item type must not be assumed to be identical.

Examples:
- `BlockType.Stone` may drop `itemID.Stone`
- `BlockType.IronOre` may drop `itemID.IronOre`
- `BlockType.Grass` may drop `itemID.Dirt`
- `BlockType.Air` should not produce a drop

When implementing mining, drops, inventory, loot tables, or block destruction, preserve this separation.

## Design rule

Do not use `itemID` as a replacement for `BlockType`.
Do not use `BlockType` as an inventory identifier unless explicitly required by an existing system.

If a conversion is needed between block and drop, use an explicit mapping layer such as:
- block settings
- loot table
- drop config
- ScriptableObject data

---
> Source: [SilverSTV/Minestable](https://github.com/SilverSTV/Minestable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
