## panopticonvr-native

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PanopticonVR Native is an **Unreal Engine 5.1** VR escape-room game developed in C++. The primary target platforms are Windows (development) and Android/Meta Quest (mobile VR via OpenXR). The project uses the VR Template as a base and extends it heavily with C++ classes that are then exposed to Blueprints.

## Build Commands

This is a standard UE5 project. Use the Unreal Build Tool (UBT) via the UE5 Editor or command line:

```bash
# Open the project in UE5 Editor (double-click or via file association)
PanVRNativeProject/PanVRNativeProject.uproject

# Build C++ from command line (adjust UnrealBuildTool path to your UE5 install)
"<UE5_ROOT>/Engine/Build/BatchFiles/Build.bat" PanVRNativeProject Win64 Development "D:/Github/PanopticonVR_Native/PanVRNativeProject/PanVRNativeProject.uproject"

# Visual Studio solution
PanVRNativeProject/PanVRNativeProject.sln
```

The module is `PanVRNativeProject` (Runtime, Default loading phase). Key UBT dependencies: `Core`, `CoreUObject`, `Engine`, `InputCore`, `EnhancedInput`, `HeadMountedDisplay`, `UMG`.

Enabled plugins: **OpenXR** (VR runtime), **ResonanceAudio** (spatial audio).

## Source Layout

```
Source/PanVRNativeProject/
├── Public/
│   ├── Core/
│   │   ├── Interface/   — IGrabInterface, IDestroyInterface, IEquipmentInitInterface
│   │   ├── Component/   — UGrabComp (grab mechanics component)
│   │   └── Debug/       — FDebugLib (DEBUG_EXEC / DEBUG_PRINTSCREEN macros)
│   ├── ActorBase/       — AVRActorBase, AVRGrabActorBase (base classes)
│   ├── VRPawn/          — AVRCharacterBase (player); AVRHand in VRPawn/Hand/
│   ├── CoreObj/         — Subsystems: VRGameInstance, VRGameInstanceSubsystem,
│   │                      VREquipmentWorldSubsystem, VRGameStateManager (stub)
│   ├── BPActorBase/     — 20+ interactive equipment actors (see below)
│   └── BPMainActorBase/ — Large environment actors: ATowerBuilding, AJailBuilding
└── Private/             — Mirrors Public structure with .cpp implementations
```

## Architecture

### Class Hierarchy

```
AActor
├── AVRActorBase (+ IIEquipmentInitInterface)
│   └── AVRGrabActorBase                        ← base for all grabbable objects
│       └── BPActorBase/* actors (equipment)
└── BPMainActorBase/* (ATowerBuilding, AJailBuilding)

ACharacter → AVRCharacterBase                   ← player pawn
AActor     → AVRHand                            ← left/right VR hand

UGameInstance        → UVRGameInstance
UGameInstanceSubsystem → UVRGameInstanceSubsystem
UWorldSubsystem      → UVREquipmentWorldSubsystem
AGameStateBase       → AVRGameStateManager
```

### Key Systems

**Grab System** — `UGrabComp` (attached to every `AVRGrabActorBase`) handles all object grabbing. Supports five modes via `EGrabType`: `Free`, `ObjToHand`, `HandToObj`, `SpawnObj`, `AutomaticObj`. `AVRHand` detects overlap via `CLGrabSphere`/`CLBox`/`CLCapsuleIndex` collision shapes and calls `GCTryGrab()` / `GCTryRelease()`. Objects implementing `IIGrabInterface` receive `OnGrabbed()` / `OnDropped()` callbacks.

**Equipment Registry** — `UVREquipmentWorldSubsystem` (World Subsystem) is the coordination hub. All `AVRActorBase` descendants call `EquipmentRegistrable()` in `BeginPlay()` to self-register. The subsystem exposes two delegates:
- `OnGameStart` — broadcast by `AAGTWLever` when pulled correctly
- `OnEBMoveOrder(FName InTag, int32 InTargetFloor)` — broadcast by `AAElevatorButton`; consumed by `ATowerBuilding`

**Audio** — Spatial audio via `HVRSoundPlayer` (declared inline in `VRActorBase.h`, not a separate file). Call `mSoundPlayer->PlaySoundEffect(this, Sound, Location)`. Global volume/pitch scaling managed by `UVRGameInstanceSubsystem` (`World_VolumeMultiplier`, `World_VolumePitch`). Plugin: Resonance Audio. Windows: 48 kHz; Android: 44.1 kHz.

**Rendering** — Forward shading, mobile-optimised (no bloom/AO/motion blur/auto-exposure). FFR at Medium. Instanced stereo + Mobile MultiView enabled. Custom depth on.

### Notable Equipment Actors (`BPActorBase/`)

| Actor | Purpose |
|---|---|
| `AAGTWLever` | Pull lever (170-180° pitch check) → broadcasts `OnGameStart` |
| `AAElevatorButton` | Multi-floor buttons (B1F/B2F/B3F) → broadcasts `OnEBMoveOrder` |
| `AASodaGun` + `AASodaBullet` | Ranged weapon with Enhanced Input mapping per hand |
| `AAFuelTank` + `AAFuelRod` | Fuel rod insertion puzzle with gauge timeline animation |
| `AACarPanel` + `AACarKey` | Multi-grab-component car puzzle with timeline key-insertion animation |
| `ATowerBuilding` | Elevator shaft; subscribes to `OnEBMoveOrder` and animates floor height |
| `AABButton`, `AAAFireAlarm`, `AAStamp`, `AALP`, `AFax` | Various puzzle interactions |
| `AACoffeeMC`, `AAToilet`, `AATurnTable` | Environmental interactive props |
| `AALighter`, `AAPaper`, `AAPunchPrinter` | Office/escape-room item interactions |
| `AAFuelRodStorage` | Storage receptacle for fuel rods |
| `AACarCrane` | Crane mechanism in car puzzle area |

### Interaction Pattern

1. `AVRHand` collision overlaps a `AVRGrabActorBase` descendant.
2. `GCTryGrab()` on the target's `UGrabComp` executes the appropriate attachment strategy.
3. `IIGrabInterface::OnGrabbed()` fires on the object (disable physics, lock constraints, etc.).
4. On release → `GCTryRelease()` → `IIGrabInterface::OnDropped()` (re-enable physics, start `DestroySelfTimer` if applicable).

### Adding New Interactive Equipment

1. Inherit from `AVRGrabActorBase` (or `AVRActorBase` for non-grabbable).
2. Implement `IIGrabInterface` if grab callbacks are needed; `IIDestroyInterface` if the object can self-destruct.
3. Call `EquipmentRegistrable()` in `BeginPlay()` (inherited via `AVRActorBase`).
4. Use `UGrabComp` (member `GC`) — do not replicate grab logic manually.
5. Use `HVRSoundPlayer` / `mSoundPlayer` for audio rather than raw `UAudioComponent`.
6. For animated interactions use `UTimelineComponent` (see `AACarPanel`, `AAFuelRod`).

### Debug Utilities (`Core/Debug/FDebugLib.h`)

- `DEBUG_EXEC(Code)` — runs `Code` only in the UE5 Editor (no-op in packaged builds).
- `DEBUG_PRINTSCREEN(Duration, Color, Content)` — on-screen message via `GEngine`; works in all configurations.

## Collision Channels

Custom channels defined in `DefaultEngine.ini`:
- `ECC_GameTraceChannel1` — `3DWidget`
- `ECC_GameTraceChannel2` — `VRPawn`
- `ECC_GameTraceChannel3` — `DontGrab`

---
> Source: [MoncageVR/PanopticonVR_Native](https://github.com/MoncageVR/PanopticonVR_Native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
