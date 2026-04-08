## 7dfps

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

7dfps is a Unity 6 (6000.2.13f1) first-person shooter game created for the 7DFPS 2025 game jam. The game features clown-themed enemies, a shooting system with ammo management, and trampoline-based physics.

## Project Structure

The Unity project is located in the `7dfps/` subdirectory. Key source code is in `7dfps/Assets/Source/`:

- **Player/** - First-person movement (WASD, jump, sprint, mouse look) and shooting mechanics with ammo/reload system
- **Actors/** - `EnemyActor` base class for enemies with health, damage handling, and death VFX
- **Clowns/** - Enemy types: `ClownCarMovement` (vehicle) and `ClownMan` (humanoid with walk/attack animations)
- **Pathfinding/** - `Rayfinder` for enemy AI movement toward player
- **Environment/** - Trampoline system (`TrampolineBase`, `TrampolineTile`, `TrampolineLimit`) for physics-based bouncing
- **VFX/** - Coroutine-based visual effects: `FlashEffect` (damage flash), `FadeEffect` (death fade), `InterpolateEffect`

## Key Architectural Patterns

- **Input System**: Uses Unity's new Input System package with `InputActionReference` bindings
- **Physics-based movement**: Player and enemies use Rigidbody physics; ground detection via `GroundedChecker` component (added at runtime)
- **Enemy AI**: Enemies use `EnemyActor` for health/damage, paired with movement components (`ClownCarMovement`, `ClownMan`) that use `Rayfinder` to navigate toward player
- **Tag-based detection**: Uses Unity tags for collision filtering ("Ground", "Bullet_Player", "Player", "WeaponMuzzle", "MainCamera")
- **Coroutine effects**: VFX components use coroutines with `WaitForSeconds` for timed effects

## Opening the Project

Open Unity Hub and add the `7dfps/` directory as a project. Unity 6 (6000.2.x) is required.

## Key Packages

- Universal Render Pipeline (URP) 17.2.0
- Input System 1.14.2
- AI Navigation 2.0.9
- Timeline 1.8.9

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mccabe93)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mccabe93)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
