## etherealmind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EtherealMind is a Minecraft Forge mod (1.20.1) that introduces COSMO - an advanced AI companion entity with dimensional storage, reality-warping abilities, and an evolving personality system.

## Common Development Commands

### Build & Run
```bash
# Build the mod
./gradlew build

# Run client
./gradlew runClient

# Run server
./gradlew runServer

# Generate data (recipes, tags, etc.)
./gradlew runData

# Clean build files
./gradlew clean
```

### Testing
```bash
# Run tests
./gradlew test

# Run specific test class
./gradlew test --tests "com.astrolabs.etherealmind.TestClassName"
```

## Project Structure

```
src/main/java/com/astrolabs/etherealmind/
├── EtherealMind.java          # Main mod class
├── common/                    # Shared code (client & server)
│   ├── entity/               # COSMO entity and related
│   ├── ai/                   # AI and personality system
│   ├── storage/              # Dimensional storage system
│   ├── network/              # Packets for client-server comm
│   └── registry/             # Item, block, entity registries
├── client/                    # Client-only code
│   ├── renderer/             # Entity and effect renderers
│   ├── particle/             # Custom particles
│   └── gui/                  # Storage and utility GUIs
└── server/                    # Server-only code
```

## Key Systems

### COSMO Entity
- Bound to individual players (one per player)
- Floating, no-collision entity with custom renderer
- Uses GeckoLib for advanced animations
- Persistent across dimensions

### AI & Personality
- Evolving personality based on playtime and interactions
- Mood system affects behavior and visuals
- Memory bank stores player interactions
- Trust level unlocks features

### Dimensional Storage
- 64 pages × 54 slots = 3,456 item slots
- Quick access bar (9 slots)
- Smart categorization with learning
- Search functionality with special syntax

## Development Guidelines

1. **Performance**: Always consider performance, especially for rendering and particles
2. **Networking**: Use packets for all client-server communication
3. **Data Persistence**: Store all important data in NBT
4. **Mod Compatibility**: Design systems to be extensible via API

## Testing in Development

When testing COSMO:
1. Use `/give @p etherealmind:cosmo_spawn_egg` to get spawn item
2. Right-click to spawn and bind COSMO
3. COSMO will follow you and respond to interactions
4. Check logs for AI state changes and debug info

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AstroLabs-MC)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AstroLabs-MC)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
