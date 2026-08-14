## geministarplatinum

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GeminiStarPlatinum is an Unreal Engine 5.6 real-time **physics simulation of the Gemini telescope and its observatory dome**. Movements are driven by physics constraints (not animation), and the longer-term intent is to mirror a live TCS (Telescope Control System) EPICS data feed. Single C++ Runtime module: `GeminiStarPlatinum`.

## Project Goals

These are the guiding objectives for the project. Weigh new work against them.

- **Immersive operator experience** — Recreate a simplified version of what telescope operators do in the command center. Users should be able to observe and interact with the live behavior of the telescope and dome, with access to the important positional data points (e.g. azimuth/elevation/Cassegrain angles, dome twist/shutter/vent state) as they move.
- **Streamlined, dual-audience data visualization** — Build a data interface that serves both laymen and engineers/operators. Support both at-a-glance world-space/camera-space figures (labels and readouts attached to the moving geometry) and detailed drill-down menus for precise numeric state. Keep the default view approachable while making the deeper data available on demand.
- **Maintainability and extensibility** — Contribute so future work doesn't require extensive re-learning. Favor clear MVC boundaries (models as source of truth, actors as views, coordinator/feed as the control source), document non-obvious physics and wiring decisions inline, and design new UI/data features so additional data points or panels can be added without rework.

## Build & Run

Workflow is **Editor + Live Coding** with a default Epic Games Launcher engine install (`C:\Program Files\Epic Games\UE_5.6`).

- **Run / open project**: open `GeminiStarPlatinum.uproject` (double-click or via Epic Launcher). If C++ binaries are stale it prompts to rebuild on launch. Default startup map is `/Game/MainWorld`.
- **Rebuild C++ while the editor is open**: **Live Coding** — `Ctrl+Alt+F11`. Prefer this for iterating on `.cpp` changes. Header/`UCLASS`/`UPROPERTY` signature changes generally require a full editor restart + rebuild.
- **Regenerate Visual Studio project files** (after adding/removing source files): right-click `GeminiStarPlatinum.uproject` → "Generate Visual Studio project files".
- **Full CLI build** (editor closed):
  ```
  "C:\Program Files\Epic Games\UE_5.6\Engine\Build\BatchFiles\Build.bat" GeminiStarPlatinumEditor Win64 Development -Project="<abs path>\GeminiStarPlatinum.uproject" -WaitMutex
  ```

There is **no automated test suite** (no Automation specs, no test target). Don't go looking for one; verify changes by running the simulation in the editor.

## Coding/Documentation style

- When commenting functions, use Doxygen-style XML comments for documentation generation purposes.
- Be highly concise and precise when describing functions and variables. Include relevant units and ranges (may point to relevant DataAsset instead if it exists)

## Git 

Do NOT stage or push commits without the express permission of the user

## Architecture — MVC (migration in progress)

The codebase is mid-refactor toward an MVC split (noted explicitly in `AssemblyModel.h`). Understanding it requires reading the model subsystems and the actors together:

- **Models** — `UAssemblyModel` (abstract) → `UTelescopeModel`, `UDomeModel`. These are `UGameInstanceSubsystem` singletons that hold target state (e.g. `AzimTarget`/`ElevTarget`/`CassTarget`, dome twist/shutter/vent targets), clamp inputs to per-axis limits, and broadcast `FOnStateChanged`. They are the source of truth.
- **Actors / Views** — `AMovingThing` (base) → `AMovingTelescope`, `AMovingDome`. Physics actors that build their component + constraint hierarchy in the constructor and, each `Tick`, read the model's targets and drive `UPhysicsConstraintComponent`s toward them. Models are fetched lazily, e.g. `GetGameInstance()->GetSubsystem<UTelescopeModel>()`.
- **Coordinator / Feed** — `UObservatoryCoordinator` (a `UGameInstanceSubsystem` holding `EControlMode { Manual, Live }`) owns a single `ULiveDataFeed` and brokers the control source: `SetControlMode(Live)` connects the feed, `Manual` disconnects it, and it broadcasts `OnControlModeChanged`. `ULiveDataFeed` is an implemented `FTickableGameObject` TCP/JSON-lines **client** that streams positional samples from an external bridge and pushes them into the model setters. Both are wired up — not stubs. See **Live data feed & TestIOC** below.

**Wiring status (important):** both actors are now wired to their models. The telescope reads `AzimTarget`/`ElevTarget`/`CassTarget` from `UTelescopeModel`; the dome reads `DomeTwistTarget`/`TopSSwingTarget`/`BotSSwingTarget`/`VentSlideTarget` from `UDomeModel`. The open/closed target values live only in `UDomeModel::SetOpen` (the previously duplicated block in `AMovingDome::Tick` has been removed), so the model is the single source of truth.

## Live data feed & TestIOC

The live feed simulates the eventual Gemini TCS connection: a three-process pipeline (`TestIOC/` EPICS soft IOC → `tools/feed_bridge/` CA→JSON translator → `ULiveDataFeed` in-engine). Unit conversion lives in the bridge; Unreal never links a CA library.

**Contract (applies even when editing only one side):** the JSON wire schema is shared between `tools/feed_bridge/data_source.py` and `ULiveDataFeed::ApplyLine` — **change both together.**

For the pipeline diagram, PV names, `pip` requirements, and the IOC → bridge → editor startup order, see the `live-data-feed` skill (`.claude/skills/live-data-feed/SKILL.md`).

## Physics conventions & gotchas

The physics tuning here is deliberate and fragile. Before changing physics behavior, understand these:

- **FluxCapacitor pattern** — both actors include a zero-mass helper body (`ParticleCube`) spun forever by a velocity drive. Its only purpose is to prevent UE's physics sleep system from halting slow/precise rotations. **Do not remove it.**
- **Velocity-then-snap drives** — `TwistComponent`/`SwingComponent`/`SlideComponent` (in `MovingThing.cpp` / `MovingDome.cpp`) apply a *velocity* drive when the error exceeds `AngularThreshold` / `LinearThreshold`, then switch to an *orientation/position spring* (snap) once within it. Angular targets are `FMath::UnwindDegrees`-clamped to [-180, 180], which intentionally limits rotation paths to mimic real mount limits.
- **Acceleration vs force mode** — `bAccelerationMode` (base default `true`, re-applied every tick) makes drives ignore mass and use acceleration units. `AMovingTelescope` overrides its Elev/Cass constraints to force mode (`SetAngularDriveAccelerationMode(false)`) because those bodies have gravity enabled.
- **Mass sensitivity** — there is a documented BUG: setting the elevation mass too high fails to stabilize. Treat masses as tuned values. Physics is configured globally in `Config/DefaultEngine.ini` under `[/Script/Engine.PhysicsSettings]` (substepping on, 16 substeps, high solver iteration counts, `bEnableEnhancedDeterminism`) and per-body in `AMovingThing::CreateMeshComponent` (sleep disabled, 50/30/100 position/velocity/projection iterations). These settings are load-bearing.
- **COM offsets** — telescope component center-of-mass offsets come from SolidWorks mass-property data (converted mm → cm). The inline comments record the source numbers.
- **Anchors** — `Base`/`Ground` bodies use mass ~`1e29` with all 6 DOF locked via `SetBaseLocked()` to act as the immovable inertial reference.

## Known incomplete / stubs

Don't assume these work — they're placeholders:

- `UAssemblyModel::ClampAndStore` — declared, not implemented in the `.cpp`, never called.
- `UMyGameInstanceSubsystem` — empty placeholder class.
- `AMovingThing::CalculateCOMOffset` — marked `FIXME`, not functional.

(`UObservatoryCoordinator::SetControlMode` and `ULiveDataFeed` are no longer stubs — both are implemented. See **Live data feed & TestIOC**.)

---
> Source: [gemini-rtsw/GeminiStarPlatinum](https://github.com/gemini-rtsw/GeminiStarPlatinum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
