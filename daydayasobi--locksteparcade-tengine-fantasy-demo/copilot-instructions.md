## locksteparcade-tengine-fantasy-demo

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

Last Updated: 2026-03-10

---

# Build & Run Commands

## Server
```bash
# Build server solution
dotnet build Server/Server.sln

# Run server
dotnet run --project Server/Main/Main.csproj

# Or open Server/Server.sln in Visual Studio/Rider and run Main project
```
Server listens on `127.0.0.1:20000` (KCP protocol) by default.

## Client
```bash
# Open in Unity Editor (Unity 2022.3.62f2 required)
# Open Client/Unity/Unity.sln for code editing
# Or open Client/Unity project directly in Unity Editor
```

## Config Generation (Luban)
```bash
# Generate config tables from Excel to Binary
# Config source: Config/Excel/
# Config output: Config/Binary/
# (Tool: Luban)
```

---

# Project Overview

This repository contains a **technical demo project** integrating:

- Unity client using **TEngine**
- .NET server using **Fantasy**
- Deterministic frame synchronization gameplay

The goal of the project is to learn and demonstrate:

- Client/server architecture
- Frame synchronization networking
- Deterministic simulation
- Fighting game style gameplay

The project is **NOT a production game**, but a learning and architecture demo.

---

# Technology Stack

Client

- Unity 2022.3.62f2
- TEngine framework
- HybridCLR (HotFix assembly)
- YooAsset (resource management)
- UniTask (async)

Server

- .NET 9
- Fantasy framework
- KCP networking
- Entitas ECS components

Tools

- Luban (Excel → config tables)
- Odin Inspector

---

# Repository Structure

## Client

Client/Unity

Important directories:

- Assets/GameScripts
- Editor
- Main
- HotFix

HotFix contains most gameplay logic.

Assets/GameScripts/HotFix

Important submodules:

- Procedure
- Net
- Battle
- Deterministic
- View

---

## Server

Server

Projects:

- Main
- Entity
- Hotfix

Responsibilities:

- **Main**: Server entry point
- **Entity**: Shared data structures and deterministic simulation
- **Hotfix**: Gameplay logic systems

---

# Deterministic Simulation (VERY IMPORTANT)

This project uses **deterministic simulation** for frame synchronization.

**ALL GAMEPLAY LOGIC MUST USE FIXED POINT MATH.**

Floating point numbers MUST NOT be used in gameplay simulation.

Do NOT use:
- `float`
- `double`
- `UnityEngine.Vector3`
- `UnityEngine.Time.deltaTime`

Instead use the deterministic math library.

---

# Deterministic Library Location

Client: `Client/Unity/Assets/GameScripts/HotFix/GameLogic/Deterministic/`

Server: `Server/Entity/Deterministic/`

The client and server must share **identical deterministic math implementations**.

The deterministic layer includes:

- `Fix64` - 64-bit fixed-point number (Q31.32)
- `FixVector2` - 2D vector using Fix64
- `FixVector3` - 3D vector using Fix64
- `FixMath` - Math utility functions

These types should be used for:
- Position
- Velocity
- Collision
- Simulation logic

---

# Gameplay Architecture

The gameplay model is **server authoritative frame synchronization**.

Basic flow:

```
Client Input → Server collects inputs → Server broadcasts frame → Clients simulate frame
```

Target simulation rate: **30 FPS** (logic), **60 FPS** (rendering with interpolation)

---

# Network Flow

## Login

Client sends: `C2G_LoginRequest` (UserName, Password)

Server responds: `G2C_LoginResponse` (UnitId, ErrorCode)

After login the player enters Lobby.

---

## Battle (Local Single-Player Frame Sync)

Current implementation is **local single-player** frame synchronization.

Key components:
- `FrameRunner` - Core logic driver at 30 FPS, Singleton implementing IUpdate
- `InputCollector` - MonoBehaviour that collects Unity input and submits to InputBuffer
- `SimulationWorld` - ECS-style simulation with entities and systems
- `RenderInterpolator` - Interpolates 30 FPS logic to 60 FPS rendering
- `InputBuffer` - Dictionary-based frame input storage (120 frames history)

---

# Client Architecture

Client uses TEngine Procedure state machine.

Procedure flow:

```
LaunchProcedure → LobbyProcedure → BattleProcedure
```

## Key Components

**FantasyManager** (`GameLogic/NetWork/FantasyManager.cs`)
- Singleton MonoBehaviour for Fantasy framework integration
- Handles Scene, Session, and Unit management
- Login functionality

**LobbyProcedure** (`GameLogic/Procedure/LobbyProcedure.cs`)
- Initializes Fantasy framework
- Shows LobbyUI
- Listens for login completion event
- Transitions to BattleProcedure after successful login

**BattleProcedure** (`GameLogic/Procedure/BattleProcedure.cs`)
- Initializes FrameRunner (Singleton)
- Creates InputCollector (MonoBehaviour for input collection)
- Creates two Player entities with deterministic components
- Creates player view GameObjects (Cubes) and binds to RenderInterpolator
- Cleans up resources on exit

**FrameRunner** (`GameLogic/Battle/FrameSync/FrameRunner.cs`)
- Singleton implementing IUpdate interface
- Core 30 FPS logic driver
- Manages SimulationWorld, InputBuffer, RenderInterpolator
- Accumulates deltaTime and executes fixed logic frames
- Calculates interpolation alpha for rendering

**SimulationWorld** (`GameLogic/Battle/Simulation/SimulationWorld.cs`)
- ECS-style world with entities and systems
- Stores player entities and frame snapshots
- Executes 4 systems in order: InputSystem, MovementSystem, StateSystem, CombatSystem
- Saves entity state snapshots for interpolation and rollback

## BattleProcedure Initialization Flow

```
BattleProcedure.OnEnter
├── Show BattleCoreUI (health bars, frame info)
├── FrameRunner.Active() - Initialize and start 30 FPS logic loop
├── Create InputCollector GameObject (collects keyboard input)
├── CreatePlayer(0, "Player1")
│   ├── Create PlayerEntity with components
│   ├── Position: (-3, 0, 0)
│   ├── Components: Position, Velocity, State, Input, Facing, Health
│   └── CreatePlayerView() - Create Cube, bind to RenderInterpolator
└── CreatePlayer(1, "Player2")
    ├── Create PlayerEntity with components
    ├── Position: (3, 0, 0)
    ├── Components: Position, Velocity, State, Input, Facing, Health
    └── CreatePlayerView() - Create Cube, bind to RenderInterpolator
```

---

# Frame Synchronization System

## Client Side Components

| Component | Location | Description |
|-----------|----------|-------------|
| `FrameRunner` | `Battle/FrameSync/` | Core 30 FPS logic driver, Singleton, implements IUpdate |
| `InputBuffer` | `Battle/Input/` | Dictionary-based frame input storage (120 frames history) |
| `InputCollector` | `Battle/Input/` | MonoBehaviour, collects Unity Input.GetKey calls, submits to FrameRunner |
| `SimulationWorld` | `Battle/Simulation/` | ECS world with entities + 4 systems |
| `RenderInterpolator` | `Battle/Render/` | Lerp between prev/current snapshots using alpha (0-1) |
| `BattleCoreUI` | `UI/BattleCoreUI/` | Displays health bars, frame info |

## FrameRunner Update Loop

FrameRunner implements IUpdate and is called by TEngine at ~60 FPS:

```
OnUpdate() [60 FPS Unity]
├── Convert Time.deltaTime to Fix64
├── accumulator += deltaTime
└── while accumulator >= FRAME_DELTA_TIME (1/30s):
    ├── UpdateLogicFrame()
    │   ├── GetFrameInput(CurrentFrame) from InputBuffer
    │   ├── World.ProcessInput(frame, input) → sets PlayerEntity.CurrentInput
    │   ├── World.Update(deltaTime) → runs 4 systems in order
    │   ├── World.SaveFrameSnapshot(frame) → stores EntityState for interpolation
    │   └── CurrentFrame++, LogicTime += FRAME_DELTA_TIME
    └── accumulator -= FRAME_DELTA_TIME
└── Calculate alpha = accumulator / FRAME_DELTA_TIME
└── RenderInterpolator.Interpolate(alpha) → Lerp entity positions for smooth rendering
```

**Key Details:**
- Logic runs at fixed 30 FPS (FRAME_DELTA_TIME = 1/30 seconds)
- Rendering interpolates between logic frames for smooth 60 FPS display
- All simulation uses Fix64 (fixed-point math) for determinism
- Frame snapshots enable rollback and client-side prediction

## Simulation Systems (executed in order each 30 FPS logic frame)

1. **InputSystem** (`Battle/Simulation/System/InputSystem.cs`)
   - Reads InputComponent and applies to VelocityComponent (horizontal movement)
   - Updates FacingComponent based on input direction
   - Triggers state transitions (Jump, Crouch, Guard, Attack)

2. **MovementSystem** (`Battle/Simulation/System/MovementSystem.cs`)
   - Applies gravity to VelocityComponent.Y when not grounded
   - Updates PositionComponent from VelocityComponent
   - Ground collision detection (Y=0)
   - Boundary clamping (X: -10 to 10)

3. **StateSystem** (`Battle/Simulation/System/StateSystem.cs`)
   - State machine transitions (Idle ↔ Walk, Jump, Attack, etc.)
   - Handles state auto-reset (attack → Idle, guard → Idle)
   - Validates state transitions based on current state

4. **CombatSystem** (`Battle/Simulation/System/CombatSystem.cs`)
   - Attack hit detection (range check + facing direction)
   - Damage calculation (Light: 5, Heavy: 10)
   - Guard damage reduction (25%)
   - Applies Hit state on damage
   - Knockdown mechanics

## Entity Components

All components implement `IComponent` marker interface.

| Component | Location | Description |
|-----------|----------|-------------|
| `PositionComponent` | `Battle/Simulation/Component/` | FixVector3 position (X, Y, Z) |
| `VelocityComponent` | `Battle/Simulation/Component/` | FixVector3 velocity (Horizontal, Y, Z) |
| `StateComponent` | `Battle/Simulation/Component/` | CharacterState + IsGrounded + state transitions |
| `FacingComponent` | `Battle/Simulation/Component/` | CharacterFacing (Left/Right) |
| `InputComponent` | `Battle/Simulation/Component/` | Current InputCommand |
| `HealthComponent` | `Battle/Simulation/Component/` | CurrentHealth/MaxHealth (Fix64, default 100) |

## State Machine

Valid state transitions:

```
Idle → Walk, Jump, Crouch, Guard, LightAttack, HeavyAttack
Walk → Idle, Jump, Guard, LightAttack, HeavyAttack
Jump → Idle (when grounded)
Crouch → Idle, Guard
Guard → Idle, Crouch
LightAttack/HeavyAttack → Idle (after attack duration)
Hit → Idle, KnockDown
KnockDown → Idle
```

**State Duration:**
- LightAttack: 5 frames
- HeavyAttack: 8 frames
- Guard: Can hold indefinitely
- Hit: 3 frames
- KnockDown: 10 frames

---

# Coding Guidelines

## Deterministic Rules

Gameplay simulation must be deterministic for frame synchronization to work correctly.

**NEVER use in simulation logic:**
- `float` / `double` - Use `Fix64` instead
- `Time.deltaTime` - Use `FrameConfig.FRAME_DELTA_TIME` instead
- `UnityEngine.Vector3` - Use `FixVector3` instead
- `Random()` without deterministic seed - Use seeded RNG only
- `UnityEngine.Physics` - Implement custom deterministic collision

**ALWAYS use:**
- `Fix64` for all numeric values in simulation
- `FixVector2` / `FixVector3` for positions and velocities
- `FixMath` utility functions for math operations
- `FrameConfig.FRAME_DELTA_TIME` for time constants

## Client/Server Logic Separation

**Unity side (rendering layer) handles:**
- Rendering and animation
- Input collection
- UI display
- Camera control

**Simulation layer (deterministic) handles:**
- Entity state and components
- Physics and collision
- Combat and damage
- State machines
- All gameplay logic

**Rule:** Never mix rendering and simulation logic. Keep them in separate layers.

## File Organization

Client simulation code structure:
```
Assets/GameScripts/HotFix/GameLogic/
├── Battle/
│   ├── FrameSync/          # FrameRunner, FrameConfig
│   ├── Input/              # InputCollector, InputBuffer, InputCommand
│   ├── Simulation/
│   │   ├── Component/      # All entity components
│   │   ├── System/         # InputSystem, MovementSystem, StateSystem, CombatSystem
│   │   └── SimulationWorld.cs
│   └── Render/             # RenderInterpolator
├── Procedure/              # LobbyProcedure, BattleProcedure
├── UI/                     # UI classes
├── Deterministic/          # Fix64, FixVector2, FixVector3, FixMath
└── NetWork/                # FantasyManager
```

---

# What Codex Should Help With

Codex can assist with:

- Network protocol design and message handlers
- Frame synchronization architecture and optimization
- Fantasy framework integration and message handlers
- Deterministic simulation systems and physics
- Code structure planning and refactoring
- ECS system design and entity management
- State machine implementation
- Input handling and buffering
- Interpolation and rendering optimization

**Codex should avoid:**
- Introducing floating point math into gameplay logic
- Mixing rendering and simulation concerns
- Creating non-deterministic code paths in simulation
- Using Unity-specific APIs in simulation layer

---

# Common Tasks

## Adding a New Simulation System

1. Create new file in `Battle/Simulation/System/`
2. Implement `ISimulationSystem` interface
3. Add `Execute(SimulationWorld world, Fix64 deltaTime)` method
4. Register in `SimulationWorld.Update()` in correct order
5. Use only Fix64 and deterministic math

## Adding a New Component

1. Create new file in `Battle/Simulation/Component/`
2. Implement `IComponent` marker interface
3. Use only Fix64 types for numeric values
4. Add to `EntityState` if needed for snapshots

## Debugging Frame Sync Issues

- Check `FrameRunner.CurrentFrame` and `LogicTime` in debugger
- Verify `InputBuffer` contains expected inputs
- Compare entity states between frames using snapshots
- Use `BattleCoreUI` to display frame info
- Enable logging in systems to trace execution order

## Performance Optimization

- Minimize allocations in hot paths (systems)
- Cache component lookups when possible
- Use object pooling for frequently created objects
- Profile with Unity Profiler to find bottlenecks
- Keep system execution order consistent

---

# Server Architecture (Fantasy Framework)

Server uses Fantasy framework for networking and ECS.

**Key Projects:**
- `Main` - Server entry point, KCP listener on port 20000
- `Entity` - Shared data structures and deterministic simulation
- `Hotfix` - Gameplay logic systems and message handlers

**Current Status:**
- Login system implemented
- Message handlers for C2G_LoginRequest / G2C_LoginResponse
- Ready for frame synchronization protocol implementation

---

# Testing & Debugging

## Local Testing

1. Start server: `dotnet run --project Server/Main/Main.csproj`
2. Open Unity Editor with Client/Unity project
3. Enter Play Mode
4. Login with any username/password
5. Battle starts with 2 local players

## Common Issues

**Frame sync not running:**
- Check `FrameRunner.Active()` is called in `BattleProcedure.OnEnter`
- Verify `InputCollector` GameObject is created
- Check TEngine `IUpdate` registration

**Determinism issues:**
- Search for `float` in simulation code
- Check for `Time.deltaTime` usage
- Verify all math uses Fix64
- Look for `Random()` calls without seed

**Input not working:**
- Check `InputCollector` is attached to GameObject
- Verify `InputBuffer.SubmitFrameInput()` is called
- Check input key mappings in `InputCommand`

---

# References

- **TEngine:** Client framework for procedures and UI
- **Fantasy:** Server framework for networking and ECS
- **FixMath.NET:** Deterministic fixed-point math library
- **YooAsset:** Resource management system
- **UniTask:** Async/await support

---

# Development Goals

## Current Milestone (In Progress)

- [x] Login system
- [x] Local frame synchronization (30 FPS)
- [x] Input collection and buffering
- [x] Player movement with deterministic physics
- [x] Basic combat system (attack hit detection)
- [x] UI display (health bars, frame info)
- [x] Render interpolation (60 FPS smooth rendering)
- [x] State machine with transitions
- [x] Guard and damage reduction mechanics

## Next Milestone

- [ ] Network frame synchronization (server broadcast)
- [ ] Room system and matchmaking
- [ ] Input broadcasting to server
- [ ] Client-side prediction and rollback
- [ ] Lag compensation

## Later Milestones

- [ ] Character animations
- [ ] Combo system
- [ ] Special moves and abilities
- [ ] Full rollback netcode
- [ ] Replay system
- [ ] Spectator mode

---
> Source: [daydayasobi/LockstepArcade-TEngine-Fantasy-Demo](https://github.com/daydayasobi/LockstepArcade-TEngine-Fantasy-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
