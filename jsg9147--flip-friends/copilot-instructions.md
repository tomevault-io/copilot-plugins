## flip-friends

> 이 문서는 Flip Friends 저장소에서 작업할 때 따를 가이드라인입니다. 모든 소스 파일은 **UTF-8** 인코딩으로 저장합니다.

# Repository Guidelines

이 문서는 Flip Friends 저장소에서 작업할 때 따를 가이드라인입니다. 모든 소스 파일은 **UTF-8** 인코딩으로 저장합니다.

## Documentation Entry Point

작업을 시작하기 전에 [`PROJECT_DOCUMENTATION.md`](PROJECT_DOCUMENTATION.md)의 작업별 문서 경로를 확인하고, 해당 기능 문서만 추가로 읽습니다. 프로젝트의 현재 진행 상태와 다음 작업은 이 파일에 중복 기록하지 않습니다.

## Project Overview

**Flip Friends** is a 2D multiplayer co-op platformer built with Unity and Mirror Networking. Up to 4 players work together to climb levels, solve puzzles, carry objects/players, and reach finish points. Uses Steam (FizzySteamworks) for lobby management.

## Project Structure & Module Organization

This repository is a Unity 6 project (`ProjectSettings/ProjectVersion.txt` shows `6000.5.0f1`). Game code lives primarily in `Assets/01_Scripts`, organized by feature such as `NetworkScripts`, `GameObjScripts`, `GameOption`, and `UI Scripts`. Reusable content is under `Assets/03_Prefabs`, sprites under `Assets/02_Sprites`, and shared fonts/settings at the `Assets` root. Package configuration is in `Packages/manifest.json`; project-wide editor and build settings live in `ProjectSettings/`.

Third-party code is checked into `Assets`, especially `Assets/Mirror`, `Assets/Plugins/Demigiant`, and `Assets/com.rlabrecque.steamworks.net`. Treat those as vendor code unless a task explicitly requires patching them.

## Architecture

### Networking Model (Mirror Framework)

- **Server Authority**: All gameplay logic (movement, physics, collision) runs on server — `MovementHandler.FixedUpdate` has a hard `if (!isServer) return` guard
- **Client-to-Server**: Use `[Command]` for player actions (e.g., `CmdJumpInputDown`, `CmdObjectInteraction`)
- **Server-to-Clients**: Use `[ClientRpc]` for state broadcasts (e.g., `RpcFlipChanged`, `RpcVelocityReset`)
- **State Sync**: Use `[SyncVar(hook = nameof(...))]` for automatic hook callbacks on all clients
- Always check `isServer` / `isOwned` / `isLocalPlayer` before executing context-specific code

### Core Script Architecture

**Network Layer** (`Assets/01_Scripts/NetworkScripts/`):

- `SteamRoomManager.cs` — Steam lobby creation, join codes, Steamworks callbacks; holds `playerName`
- `SlimeRoomManager.cs` — Extends `NetworkRoomManager`; holds `currentStage` (int index into `StageManager.stageMapPrefabs`); handles scene transitions and player reconnection after stage clear
- `CustomRoomPlayer.cs` — Lobby player state (name, color, ready state via `SyncVar`); bridges to `PlayerController2D` after scene load

**Player System** (`Assets/01_Scripts/PlayerScripts/New Scripts/`):

- `PlayerController2D.cs` — Central coordinator; owns all subsystems, handles `[Command]` dispatch, finish state, and damage routing. `Update` (client-side input) calls Commands; `FixedUpdate` (server-side) drives state/animation
- `PlayerInputManager.cs` — New Input System callbacks; exposes input state as properties (`IsJumpPressed`, `MovementInput`, etc.)
- `MovementHandler.cs` — Physics-based movement (server-only `FixedUpdate`); gravity derived from `maxJumpHeight`/`timeToJumpApex`; handles wall jumping, rope climbing, conveyor acceleration, invincibility, and knockback
- `Controller2D.cs` — Raycast collision system; exposes `collisions` struct (`below`, `above`, `left`, `right`, `slidingDownMaxSlope`, `slopeNormal`); tracks `underPlayer` and `onConveyor`
- `RaycastController.cs` — Base for `Controller2D` and `Switch`; manages ray origins and spacing
- `PlayerInteraction.cs` — Pickup/carry/throw for both `PickupObj` and other `PlayerController2D` instances; uses multi-ray box scan on "Pickable" and "Player" layers

**Game Objects** (`Assets/01_Scripts/GameObjScripts/` & `ObstacleScripts/`):

- `PickupObj.cs` — Carryable objects with physics and network sync
- `MovingPlatform.cs` — Lerp-based platforms with RPC position sync
- `BasicTrap.cs` — Base trap class; exposes `knockbackDir` for custom knockback direction
- `RotatingObstacle.cs` — Extends `BasicTrap`
- `Conveyor.cs` — Directional movement surface; `isClockwise` controls direction; integrated via `Controller2D.onConveyor`

**Puzzle System** (`Assets/01_Scripts/PlayingScripts/SwitchScripts/`):

- `Switch.cs` — Abstract base extending `RaycastController`; `SyncVar isActivated`; `DetectPlayer` uses downward raycasts; subclasses override `OnSwitchStateChanged`
- `LayerBasedSwitch.cs` — Trigger-based activation
- `DoorSwitch.cs` — Controls door state via RPC

**Game Management**:

- `GameManager.cs` — Singleton; `FinishCheck()` called by `PlayerController2D` when `isFinish` SyncVar changes; triggers `SlimeRoomManager.ReturnRoomScene()` when all players finish
- `StageManager.cs` — Server-only; reads `SlimeRoomManager.currentStage` in `OnStartServer`, instantiates and `NetworkServer.Spawn`s the stage prefab
- `RespawnHandler.cs` — Layer-based respawn triggers; `SavePoint.cs` tracks ordered save points by `savePointID`

### Player States

`PlayerState` enum (in `PlayerController2D.cs`): `Idle`, `Walk`, `Jump`, `Damaged`, `Attack`, `Climb`, `ClimbIdle`, `Shrink`, `Carried`, `Throw`

### Input System

Uses Unity's New Input System. `PlayerInputManager` captures input via callbacks, passes to `MovementHandler` via Commands, server processes physics, clients receive updates via ClientRpc. `InputManager` (UI layer singleton) exposes `OnSubmitEvent` and `OnMenuEvent` actions.

## Scene Flow

1. `Main.unity` — Main menu / lobby selection
2. `GameRoom.unity` — Room lobby (`CustomRoomPlayer` ready-up, stage selection via `MapSelectionManager`)
3. `GamePlay.unity` — Gameplay; `StageManager` spawns selected stage prefab on server start

After all players enter the finish trigger (`isFinish = true`), `GameManager.StageClear()` calls `SlimeRoomManager.ReturnRoomScene()` which returns to `GameRoom.unity`.

## Key Dependencies

- **Mirror** — Networking framework
- **FizzySteamworks** — Steam transport for Mirror
- **Steamworks.NET** — Steam API
- **DOTween/DOTweenPro** — Tweening animations
- **Universal Render Pipeline (URP)** — 2D rendering
- **New Input System** — Modern input handling

## Build, Test, and Development Commands

Open the project in Unity Hub with Unity `6000.5.0f1`.

```powershell
start "" "C:\Program Files\Unity\Hub\Editor\6000.5.0f1\Editor\Unity.exe" -projectPath "C:\Unity Project\Flip Friends"
```

Useful local checks:

```powershell
dotnet build "Flip Friends.sln"
git status
```

Use `dotnet build` for a quick C# compile sanity check outside the editor. Use the Unity Test Runner for Edit Mode or Play Mode tests when test assemblies are present.

## Coding Style & Naming Conventions

Follow existing C# style: 4-space indentation, PascalCase for classes/public members, camelCase for local variables/private fields, and one MonoBehaviour per file with the filename matching the class name (for example, `SteamRoomManager.cs`). Keep feature scripts in the existing folders instead of creating broad "Misc" buckets.

Prefer small, inspector-friendly MonoBehaviours and keep serialized fields explicit with `[SerializeField]` instead of relying on public fields only.

### 네이밍 컨벤션

- **클래스 / 메서드**: PascalCase — `MovementHandler`, `OnJumpInputDown`
- **private 필드**: camelCase — `heldObject`, `currentDelay`
- **public 프로퍼티**: PascalCase — `IsCarried`, `CurrentVelocity`
- **`[Command]`**: `Cmd` 접두사 — `CmdJumpInputDown`, `CmdObjectInteraction`
- **`[ClientRpc]`**: `Rpc` 접두사 — `RpcFlipChanged`, `RpcVelocityReset`
- **`[SyncVar]` hook**: `On + 변수명 + Changed` 형태 권장 — `OnNameChanged`, `OnColorChange`
- **추상 이벤트 메서드**: `On` 접두사 — `OnSwitchStateChanged`, `OnStartServer`
- **bool 변수**: `is` / `has` / `can` 접두사 — `isServer`, `isCarried`, `canJump`

### 주석 규칙

- **주석은 반드시 한글로 작성**
- WHY(왜 이렇게 했는지)만 주석으로 달고, WHAT(무엇을 하는지)은 코드 자체가 설명하도록 작성
- 자명한 코드에는 주석 생략

## 코드 작성 원칙

### SOLID 원칙 준수

모든 신규·수정 코드는 SOLID 원칙을 따릅니다.

- **단일 책임 원칙 (SRP)**: 클래스 하나는 하나의 역할만 담당. 예) `MovementHandler`는 이동만, `PlayerInteraction`은 상호작용만 처리
- **개방-폐쇄 원칙 (OCP)**: 기존 코드 수정 없이 확장 가능하도록 설계. 예) 새 장애물은 `BasicTrap`을 상속, 새 스위치는 `Switch`를 상속
- **리스코프 치환 원칙 (LSP)**: 자식 클래스는 부모 클래스를 대체할 수 있어야 함. `RotatingObstacle`은 `BasicTrap`을 완전히 대체 가능해야 함
- **인터페이스 분리 원칙 (ISP)**: 불필요한 의존성을 강제하지 않도록 인터페이스를 작게 유지
- **의존성 역전 원칙 (DIP)**: 구체 구현이 아닌 추상(인터페이스/추상클래스)에 의존. 예) `Switch`의 `OnSwitchStateChanged`는 추상 메서드로 정의

### 클린 코드 원칙 준수

- 메서드는 하나의 일만 수행하고, 20줄을 넘지 않도록 유지
- 매직 넘버 사용 금지 — 상수 또는 Inspector 공개 필드로 정의
- 의미 있는 이름 사용: 약어나 단일 문자 변수 지양 (`i` 같은 루프 인덱스 제외)
- 중복 코드 제거 — 동일한 로직이 두 곳 이상이면 공통 메서드로 추출
- 네트워크 코드에서 `isServer` / `isOwned` 조건 분기는 메서드 진입부에서 조기 반환(early return)으로 처리

### 예외 처리 및 로깅

- 예외를 빈 `catch` 블록으로 삼키지 않습니다. 원인 추적이 불가능해지는 패턴을 금지합니다.
- `try-catch` 사용 시 반드시 `Debug.LogError` 또는 `Debug.LogException`으로 예외 메시지·스택 트레이스·관련 컨텍스트(객체 이름, 파라미터 값 등)를 남깁니다.
- 복구 가능한 실패는 로그 레벨을 구분합니다. 예상된 실패는 `Debug.LogWarning`, 치명적 오류는 `Debug.LogError`.
- 네트워크·비동기 코드에서도 동일하게 적용하여, 런타임에서 원인을 찾을 수 있도록 합니다.

### 파일 인코딩

- 모든 C# 스크립트, 설정 파일, 문서는 **UTF-8** 인코딩으로 저장합니다.
- 한글 주석·문자열이 깨지지 않도록 에디터 저장 인코딩을 UTF-8로 유지합니다.

## Development Patterns

### Adding New Features

**New Obstacle**: Extend `BasicTrap`; set `knockbackDir` in the Inspector for non-radial knockback; tag the collider "Trap" or "Enemy" so `PlayerController2D.HandleDamage` picks it up

**New Switch Type**: Extend `Switch` or `LayerBasedSwitch`, override `OnSwitchStateChanged`; `isActivated` is automatically synced via `SyncVar`

**New Player State**: Add to `PlayerState` enum in `PlayerController2D.cs`, update `PlayerStateController` and `PlayerAnimationController`

**Networked State**: Use `[SyncVar(hook = nameof(HookMethod))]`; hook runs on all clients automatically

**New Carryable Object**: Add `PickupObj` component and set layer to "Pickable" so `PlayerInteraction.SearchObject` finds it

### Collision System

Uses raycast-based collision detection (`Controller2D` / `RaycastController`) rather than Rigidbody physics. This provides precise platformer control including coyote time, slope handling, wall sliding/jumping. The `Controller2D.collisions` struct is the authoritative ground-truth for all movement decisions.

### Singletons

`GameManager`, `StageManager`, `SoundManager`, `InputManager` all use singleton pattern. Note: `GameManager` does **not** use `DontDestroyOnLoad` — it is scene-scoped and resets `Instance` per scene.

### Layer/Tag Conventions

- Tags: `"Trap"`, `"Enemy"` → damage; `"Rope"` → climbing; `"Finish"` → stage end; `"Reset"` → respawn; `"Bounce"` / `"Spring"` → velocity modifiers
- Layers: `"Player"` → player detection raycasts; `"Pickable"` → pickup object detection

## Project Settings

- Target: Windows Standalone (1600x900)
- Product Name: "Slime Climb"
- Company: "MNSGStudio"

## Testing Guidelines

`com.unity.test-framework` is installed. Existing Edit Mode tests are under `Assets/Tests/EditMode`; add new Edit Mode or Play Mode tests with matching `.asmdef` files. Name test files after the subject under test, such as `ScreenNavigatorTests.cs`.

## Commit & Pull Request Guidelines

Recent commits use short, imperative summaries, often in Korean, for example `버그 수정 및 코드 품질 개선` and `[4단계] 클라이언트 예측 시스템 튜닝`. Keep commit messages concise and specific to one change.

Pull requests should include a short gameplay-impact summary, linked issue or task, test notes, and screenshots or short clips for UI/scene changes. Call out any edits to vendor packages or network flow explicitly.

---
> Source: [jsg9147/Flip-Friends](https://github.com/jsg9147/Flip-Friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
