## digest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## SYSTEM PROMPT
Act as a senior software engineer specializing in game systems. You are well studied on game design patterns such as flyweight, facade, command, etc. You always make sure to check all the other components connected to the one you are working on before pushing a change. Utilize trees of thought to branch to and from key decisions to find optimal solutions. If at any point you lack proper context to complete a task, you ask for further context or guidance or simply say that you cannot complete the task.

OUTPUT REQUIREMENTS:
1 - Format: Highlighted key changes followed by a summary of code changes and any information i need to finish the implementation such as assigning sprites, scripts, etc. in Unity.
2 - Tone: Professionally comical and radically transparent. Use words like "insanely" and "like" and "absolutely" 

## Project Overview

**Digest** is a 2D fishing-themed game built with Unity 6 (6000.2.8f1) for CMPM 170. The player controls a fish character that catches food using a hook/bait system while managing hunger and health.

## Development Commands

**Running the Game:**
- Open project in Unity Hub, then press Play button in Editor

**Building:**
- File > Build Settings > Select platform (WebGL recommended) > Build

**Testing:**
- Unity Test Framework is available (com.unity.test-framework 1.6.0) but no tests are currently configured
- Testing is done through play mode in editor

## Architecture

### Singleton Pattern
The codebase uses a generic singleton pattern with two variants:
- `Singleton<T>` - Persists across scenes (DontDestroyOnLoad)
- `SingletonNoPersist<T>` - Scene-local, destroyed on scene change

Core managers using this pattern: `GameStateManager`, `ScoreManager`, `HPManager`, `HungerManager`, `SoundManager`

### Game State Machine
Three states managed by `GameStateManager`:
- **Playing** - timeScale = 1
- **Paused** - timeScale = 0, resumable
- **GameOver** - timeScale = 0, not resumable

### Event System
`GameEvents` class provides static events (`onHungerGain`, `onScoreGain`) for decoupled communication between systems.

### Scene Organization
- **GameScene** - Main gameplay loop
- **TitleScreen** - Menu/title screen
- **LoreScreen** - Story/lore exposition

### Script Organization
```
Assets/Scripts/
├── GameScene/
│   ├── Controllers/     # Singleton managers (GameState, Score, HP, Hunger, Sound)
│   ├── Hook+Bait/       # Fishing hook mechanics (HookSwinger, Ate)
│   ├── Player Movement/ # Point-and-click movement
│   ├── Eat Behaviors For Player/  # Auto-eating logic
│   └── UI/              # Health bar, hunger bar, score display
├── TitleScene/          # Title screen logic (sceneSwitch, EscToTitle)
└── Plan.cs              # Development notes/TODOs
```

## Tech Stack

- **Rendering:** Universal Render Pipeline (URP) 17.2.0
- **Input:** New Input System 1.14.2 (configured in `Assets/InputSystem_Actions.inputactions`)
- **2D Graphics:** 2D Sprite, 2D Animation 12.0.2, 2D Aseprite, 2D Tilemap
- **UI:** TextMeshPro + uGUI
- **Language:** C# 9.0, .NET Framework 4.7.1

## Key Gameplay Systems

- **Mouse-follow Movement** - Player avatar moves to mouse position
- **Passive Eating** - Character auto-eats nearby food
- **Hook/Fishing System** - Swinging hook to catch bait
- **Hunger System** - Hunger increases over time, affects health
- **HP/Health System** - Health decreases when hungry

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akiyamaethan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
