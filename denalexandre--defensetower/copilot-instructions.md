## defensetower

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DefenseTower is a cross-platform tower defense game built with .NET MAUI and custom 2D canvas rendering. The UI text is in French.

## Build & Run Commands

```bash
# Build (Windows)
dotnet build DefenseTower/DefenseTowerOrigin.csproj -f net10.0-windows10.0.19041.0

# Run (Windows)
dotnet run --project DefenseTower/DefenseTowerOrigin.csproj -f net10.0-windows10.0.19041.0

# Build all platforms
dotnet build

# Run via Aspire host
dotnet run --project DefenseTower.AppHost/DefenseTower.AppHost.csproj
```

There are no tests in the project currently.

## Architecture

**Solution structure:** Three projects in `DefenseTower.slnx`:
- `DefenseTower/` — Main MAUI game app
- `DefenseTower.AppHost/` — .NET Aspire orchestration host
- `DefenseTower.ServiceDefaults/` — Shared service extensions (telemetry, health checks)
- `Archive/` — Legacy Python prototype (not part of the build)

**Game loop and rendering:** `GameCanvasView.cs` is the central file (~1500 lines). It contains:
- A `GameDrawable` class implementing `IDrawable` — the main draw/update loop
- 60 FPS timer-driven updates via `IDispatcherTimer`
- Virtual coordinate system (1080x700) that scales to actual screen size
- Game state machine: `Stopped` → `Playing` → `Paused`
- A 130px HUD overlay strip at the top of the canvas

**Domain models** (in `Models/`):
- `Tower.cs` — Tower types (archer, mitrailleuse, canon), targeting, firing cooldowns, drawing
- `Enemy.cs` — Enemy types with damage resistances, path interpolation movement
- `Projectile.cs` — Projectile trajectory, collision detection
- `Tree.cs` — Candy trees (resource generators)
- `DamageType.cs` — Enums for DamageType and EnemyType

**Rendering pipeline order:** background → path → entities → projectiles → UI overlay. Canvas state is saved/restored around scale transforms.

## Key Patterns

- Each model owns both its logic (`Update()`) and its rendering (`Draw()`) methods
- Tower targeting: find closest enemy within range using distance checks
- Enemy movement: path segment interpolation with pre-computed waypoints
- Damage system: type-based resistance multipliers per enemy type
- Resource system: candy trees generate currency over time

## Configuration

- Target framework: .NET 10.0 (multi-target: Windows, Android, iOS, macOS)
- Build configurations: Debug, Release, WindowsMachine
- Nullable reference types and implicit usings are enabled

---
> Source: [DenAlexandre/DefenseTower](https://github.com/DenAlexandre/DefenseTower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
