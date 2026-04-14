## cozygrove-mods

> **IMPORTANT:** Before working on this project, you MUST query the Mempalace knowledge graph to retrieve relevant context about this codebase.

# Cozy Grove Mods - All agent Context Guide

## Mempalace Integration

**IMPORTANT:** Before working on this project, you MUST query the Mempalace knowledge graph to retrieve relevant context about this codebase.

### Always Do This First

1. Use `mempalace_kg_query` to query for relevant entities (e.g., "CozyGrove", "AutoHarvest", "AutoFishing")
2. Use `mempalace_search` to find relevant documentation and code patterns
3. Use `mempalace_traverse` to explore connected topics

### Wing Structure

- **cozy-grove-mods**: Main project wing
  - `game-interfaces`: Game API classes from Assembly-CSharp (dump.cs)
  - `mod-development`: Mod development patterns and utilities
  - `auto-harvest`: Auto-harvest mod specifics
  - `auto-fishing`: Auto-fishing mod specifics
  - `auto-net`: Auto-netting mod specifics

### When to Update Context

- After completing a significant feature or fix
- When discovering new game interface classes
- When creating a new mod project
- When learning something non-obvious about the codebase

### How to Update Context

Use `mempalace_add_drawer` to add new knowledge:
```json
{
  "wing": "cozy-grove-mods",
  "room": "mod-development",
  "content": "New discovery here...",
  "source_file": "relevant file path"
}
```

Use `mempalace_kg_add` to add facts to the knowledge graph:
```json
{
  "subject": "AutoHarvest",
  "predicate": "uses",
  "object": "PlayerInventory"
}
```

## Project Structure

This is a modding framework for the game Cozy Grove. Each mod is typically a separate project under `projects/`.

## Game Interface Reference

See `resources/dump.cs` for the decompiled game assembly. Key interfaces are documented in the Mempalace under `cozy-grove-mods/game-interfaces`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/007lock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
