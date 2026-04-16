## bugs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Unity 6000.2.2f1 project implementing a grid-based bug collection game. The player moves on a grid to collect bug clouds while avoiding traps. The game features:

- Grid-based movement system using Unity's New Input System
- Bug cloud spawning with Manhattan distance constraints
- Dynamic trap placement avoiding optimal paths
- Pathfinding visualization for optimal collection routes

## Key Architecture Components

### Core Game Systems

- **GridMover**: Handles player movement on a discrete grid using Input System (keyboard: WASD/Arrow keys)
- **BugCloudSpawner**: Spawns bug clouds at optimal distances from player using Manhattan distance calculations
- **TrapSpawner**: Places traps randomly while avoiding player, bug clouds, and optimal path positions
- **BestPath**: Calculates and visualizes optimal collection routes between player and bug clouds
- **PlayerCollector**: Handles collision detection for bug collection

### Grid System Design

The game uses a unified grid system where:
- Grid origin is at (0,0) with 1-unit cell size
- All entities snap to integer coordinates
- Movement is constrained to cardinal directions
- Spawning systems use Vector2Int for grid calculations

### Scene Structure

- Main scene: `Assets/Scenes/SampleScene.unity`
- All game scripts are in `Assets/Scripts/`
- Uses Unity prefabs for instantiation of game objects

## Common Development Tasks

### Building the Project
Unity projects are built through the Unity Editor. Open the project in Unity Editor 6000.2.2f1 and use Build Settings (File > Build Settings).

### Code Style
- Scripts are written in C#
- Uses French comments and debug logs
- MonoBehaviour lifecycle methods (Start, Awake, Update)
- Public fields for Inspector configuration with [Header] and [Tooltip] attributes

### Testing
The project includes Unity Test Framework (`com.unity.test-framework`: `1.5.1`) in the package manifest.

### Key Dependencies
- `com.unity.inputsystem`: `1.14.2` - New Input System for player controls
- `com.unity.render-pipelines.universal`: `17.2.0` - URP rendering
- `com.unity.ai.navigation`: `2.0.8` - Navigation mesh features

## Important Implementation Notes

### Spawning Logic
- BugCloudSpawner runs in Awake() to ensure clouds exist before TrapSpawner's Start()
- TrapSpawner queries existing BugCloud and BestPath tagged objects to avoid placement conflicts
- Manhattan distance calculations ensure proper spacing between game elements

### Movement System
- GridMoverNewInput uses wasPressedThisFrame for discrete step-based movement
- Coroutine-based smooth interpolation between grid positions
- Optional tile validation using raycasting

### Path Visualization
- BestPath generates visual indicators for optimal collection routes
- Instantiates prefabs at ground level (Y=0.01f) to show path
- Uses the same prefab as bug clouds for path visualization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PierreChagnon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
