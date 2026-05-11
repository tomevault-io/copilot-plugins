## tamal-mod-fabric1-21-1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

Build an epic boss mob called **"Zipacná, the Mountain Eater"** for the **Tamaland** Minecraft server. This is a Fabric 1.21.1 mod using GeckoLib 4 for animations.

## Build Commands


```bash
./gradlew build          # Compile and package mod → build/libs/
./gradlew runClient      # Launch dev Minecraft client
./gradlew runServer      # Launch dev Minecraft server
./gradlew genSources     # Generate decompiled source mappings
```

Built artifact goes to `build/libs/`. No test framework is configured.

## Stack

- **Java 21**, Fabric Loader 0.19.2, Minecraft 1.21.1
- **GeckoLib 4** for entity models and animations (must be added as dependency)
- **Yarn mappings** (`1.21.1+build.3`) — always use Yarn names, not MCP/Mojang
- **Fabric Loom 1.16-SNAPSHOT** as the build plugin

## Architecture

Entry points (defined in `fabric.mod.json`):
- `TamalMod` (`ModInitializer`) — common/server-side registration
- `TamalModClient` (`ClientModInitializer`) — client-side registration (renderers, GeckoLib)
- `TamalModDataGenerator` (`DataGeneratorEntrypoint`) — recipe/loot/tag generation

Mixins are in `net.ramos.tamalmod.mixin`, configured via `tamal-mod.mixins.json`. Use `@Inject` sparingly; prefer Fabric API event hooks when available.

**MOD_ID:** `"tamal-mod"` (defined as constant in `TamalMod.java`)

## Zipacná Boss Design

### Entity Architecture
- Use **Interaction Entities** for multi-part hitboxes (head = critical, 3× projectile damage; legs = tank/stun zone)
- Head hitbox stun: if legs take X damage, Zipacná lowers its head for 5 seconds
- Base Zipacná on a custom `LivingEntity` (not a scaling colossus — terrestrial giant)
- Minion `MayanVultureEntity` inherits from `PhantomEntity`, adapted for low-flight harassment AI

### Attacks
1. **Sonic Roar** — vector-based push + particle burst; no heavy physics
2. **Obsidian Spike Eruption** — AOE area detection to spawn temporary block/entity spikes

### Phases (Boss Bar required)
- 75% HP → spawn wave of Maya Vultures
- 50% HP → spawn wave of Maya Vultures
- 25% HP → spawn wave of Maya Vultures
- Death → drop "Corazón de la Tierra" (Jade item)

## M1 Optimization Rules (8GB RAM target)

- Use AOE vector/area detection instead of per-entity physics calculations
- Send particle packets **only to nearby clients** (`world.getPlayers()` with distance check)
- Ensure all temporary entities (spike entities, interaction hitboxes) clean themselves up on boss death or despawn
- Monitor MSPT; boss tick logic must be lightweight

## Code Style

- Modern Java 21: use Records where appropriate, pattern matching, sealed interfaces if modeling state machines
- Explain Computer Science concepts applied (state machines, vectors, AOE) in brief inline comments when non-obvious
- All code must be compatible with **Minecraft 1.21.1** and **GeckoLib 4** latest syntax
- GeckoLib renderer registration happens client-side in `TamalModClient.onInitializeClient()`

---
> Source: [R4M0S17/tamal-mod-fabric1.21.1](https://github.com/R4M0S17/tamal-mod-fabric1.21.1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
