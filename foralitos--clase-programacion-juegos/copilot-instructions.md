## clase-programacion-juegos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Unity 6 (6000.0.57f1) 3D game project using the Universal Render Pipeline (URP). The project features a simple player controller with movement and shooting mechanics.

## Key Technologies

- **Unity Version**: 6000.0.57f1
- **Render Pipeline**: Universal Render Pipeline (URP) 17.0.4
- **Input System**: New Input System 1.14.2
- **Physics**: Unity Physics with Rigidbody components
- **Custom Package**: Unity MCP Bridge (com.coplaydev.unity-mcp)

## Project Structure

```
Assets/
├── Scenes/              # Game scenes
│   └── SampleScene.unity
├── scripts/             # Player gameplay scripts
│   ├── PlayerMovement.cs
│   └── PlayerShooting.cs
├── materials/           # Material assets (IA1-3, pj)
├── Settings/            # URP render pipeline settings
│   ├── Mobile_RPAsset.asset
│   ├── PC_RPAsset.asset
│   └── *_Renderer.asset files
└── TutorialInfo/        # Template documentation

Packages/
└── manifest.json        # Package dependencies
```

## Core Gameplay Systems

### Player Movement (`PlayerMovement.cs`)
- Physics-based movement using Rigidbody with `linearVelocity`
- Boundary constraint system limiting player to defined area (±9 units)
- Rotation frozen to prevent tilting
- Input via standard Unity Input axes (Horizontal/Vertical)

### Player Shooting (`PlayerShooting.cs`)
- Fire rate limited shooting system with directional aiming
- Cardinal direction targeting (N/S/E/W) based on last movement input
- Dynamically creates bullet primitives (spheres) at runtime
- Bullets destroy enemies on collision via `EnemyMovement.TakeDamage()`
- Automatic cleanup via `BulletLifetime` component
- Input via mouse button or spacebar

### Enemy System
**`EnemyMovement.cs`**
- AI follows player using target-seeking behavior
- Physics-based movement on XZ plane only
- Health system with damage handling
- Auto-destruction on player collision or boundary exit (±12 units)
- Requires GameObject tagged "Player" to function

**`EnemySpawner.cs`**
- Manages enemy population with configurable max count
- Spawns from random cardinal directions at defined distance
- Supports multiple enemy prefab variants
- Automatic cleanup of destroyed enemy references
- Gizmo visualization for spawn area in editor

## Development Commands

### Opening in Unity
Open this folder in Unity Hub or launch Unity 6000.0.57f1 directly with this project path.

### Testing in Unity
1. Open `Assets/Scenes/SampleScene.unity`
2. Press Play in Unity Editor
3. Use WASD/Arrow keys to move
4. Use Mouse Click or Spacebar to shoot

### Code Editing
The project uses Unity's default C# scripting workflow. Scripts are compiled automatically by Unity when modified.

## Important Implementation Details

### Physics Notes
- Player uses `linearVelocity` (Unity 6 API) instead of deprecated `velocity`
- Movement boundaries are enforced through position clamping and velocity zeroing
- Bullets use combined forward and upward velocity for arc trajectory

### Object Lifecycle
- Bullets are created dynamically via `GameObject.CreatePrimitive()`
- Automatic cleanup through `Destroy()` timer and collision detection
- Bullet destruction skips player collisions via tag check
- Enemies spawned via `Instantiate()` with automatic reference tracking
- Enemy cleanup on death, collision with player, or boundary exit

### Input System
Project includes both:
- Traditional Input Manager (currently used in scripts)
- New Input System package (available but not yet integrated)

## Common Tasks

### Adding New Scripts
Place C# scripts in `Assets/scripts/` directory following existing naming convention (PascalCase).

### Modifying Gameplay Behavior
- **Player Movement**: Edit `PlayerMovement.cs` - adjust `moveSpeed` and `boundarySize` values
- **Player Shooting**: Edit `PlayerShooting.cs` - modify `bulletSpeed`, `fireRate`, and `bulletLifetime`
- **Enemy AI**: Edit `EnemyMovement.cs` - tune `moveSpeed`, `health`, and `boundaryLimit`
- **Enemy Spawning**: Edit `EnemySpawner.cs` - configure `maxEnemies`, `spawnInterval`, and `spawnDistance`
- **Physics**: Adjust Rigidbody constraints and velocity calculations in respective scripts

### Working with Materials
Materials are stored in `Assets/materials/`. Current materials: IA1, IA2, IA3 (enemies), pj (player). Assign via Inspector or script using `Renderer.material`.

### Setting Up Enemy Prefabs
1. Create enemy GameObject with required components: Rigidbody, Collider, EnemyMovement script
2. Assign material from `Assets/materials/`
3. Tag as appropriate (not "Player")
4. Save as prefab and assign to EnemySpawner's `enemyPrefabs` array

## Unity MCP Integration

This project includes the Unity MCP Bridge package, enabling Model Context Protocol integration for enhanced AI-assisted development. This allows Claude Code to interact with Unity Editor state, scenes, and GameObjects through MCP tools.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Foralitos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Foralitos)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
