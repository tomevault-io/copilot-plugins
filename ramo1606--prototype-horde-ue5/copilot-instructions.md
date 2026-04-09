## prototype-horde-ue5

> Third-person shooter survival game. Scope similar to Boxhead 2Play but with

# TPS Survival Game — Project Context for Claude

## Project Overview

Third-person shooter survival game. Scope similar to Boxhead 2Play but with
a free TPS camera (Uncharted / Zombie Army 4 style) with over-the-shoulder
aiming. Wave-based enemy survival with score and combo multiplier.

**Engine**: Unreal Engine 5  
**Language**: C++ (core systems) + Blueprint (high-level logic and iteration)  
**Target platforms**: Android (Adreno 610), Switch 2, PC/Steam  
**Performance floor**: Adreno 610 — all systems must be validated on this target  
**FPS targets**: 30fps mobile · 60fps Switch 2 and PC  

---

## Art Direction

**Assets**: Kay Lousberg — Prototype Bits + Character Animations (CC0)  
**Rig**: Rig_Medium for player and standard enemies  
**Style**: Cel shading with quantized NdotL (2 levels) + outline via Custom Depth  
**Texture**: Single gradient atlas (128x128 on mobile, 1024x1024 source)  
**Palette**: Defined in Fase 0 — all assets use the project master material  

Do NOT suggest high-poly assets, PBR workflows, normal maps, or specular
complexity. The visual pipeline is intentionally minimal.

---

## Rendering Pipeline

- **Renderer**: Mobile Forward Renderer (`r.Mobile.ForwardShading=1`)
- **GI**: None — baked Lightmass only, no Lumen
- **Shadows**: Baked on mobile · Shadow Maps on Switch 2 and PC
- **AA**: FXAA on mobile · TAA on Switch 2 and PC
- **Disabled**: Lumen, Nanite, VSM, Motion Blur, Bloom (or minimal), SSR, SSAO
- **Post process**: Outline pass (Custom Depth stencil) + LUT color grading only
- **Scalability**: Three profiles — Mobile / Switch2 / PC via DefaultScalability.ini

---

## Architecture

### Core Systems (C++)
- `ACharacter` — player character, extends ACharacter
- `AEnemyCharacter` — enemy base class
- `UHealthComponent` — reusable health/damage component
- `AWeapon` — weapon base class, child classes per weapon type
- `AWaveManager` — wave sequencing and spawn logic
- `AGameMode` — game loop, score, combo multiplier
- `UCameraComponent` — spring arm parameter interpolation for explore/aim states

### Camera States
- **Exploration**: SpringArm length ~400, locomotion-relative orientation, FoV 75-80
- **Aim (over-shoulder)**: SpringArm length ~150, offset X:30 Y:60, aim-relative orientation, FoV 65
- Transition time: ~0.2s interpolation in Tick

### Input
- Enhanced Input System with two mapping contexts: Gameplay and UI
- Actions: Move, LookAround, Aim (hold), Fire, Reload, SwitchWeapon, Pause

### AI
- Behavior Tree per enemy type
- Blackboard keys: TargetActor, CanSeePlayer, DistanceToTarget, IsStaggered
- Perception: Sight (radius/angle) + Hearing
- States: Idle/Patrol → Chase → Attack → Stagger → Death
- NavMesh update frequency tuned for mobile (not default)

---

## Weapons

| Weapon     | Type      | Notes                              |
|------------|-----------|------------------------------------|
| Pistol     | Hitscan   | Base weapon, always available      |
| Shotgun    | Hitscan multi-pellet | Short range, spread  |
| SMG        | Hitscan auto | High cadence, incremental spread |

Pickup system: weapons spawn in map as pickups, auto-switch or manual.

---

## Project Structure

```
Content/
│
├── Core/                      — Assets del engine del juego
│   ├── GameMode/              — BP_GameMode, BP_GameState
│   ├── PlayerController/      — BP_PlayerController
│   ├── GameInstance/          — BP_GameInstance
│   └── Input/                 — InputMappingContext, InputActions
│
├── Character/
│   ├── Player/
│   │   ├── Mesh/              — SK_Player, SK_Player_Physics
│   │   ├── Animations/        — ABP_Player, BlendSpaces, AimOffset
│   │   └── Materials/         — MI_Player_Base, MI_Player_Alt
│   └── Enemies/
│       ├── Standard/          — Mesh, ABP, Materials del enemigo base
│       └── Heavy/             — Mesh, ABP, Materials del enemigo pesado
│
├── Weapons/
│   ├── Shared/                — Meshes de impacto, decals, sonidos genéricos
│   ├── Pistol/                — SK_Pistol, SFX, VFX, DA_PistolData
│   ├── Shotgun/               — SK_Shotgun, SFX, VFX, DA_ShotgunData
│   └── SMG/                   — SK_SMG, SFX, VFX, DA_SMGData
│
├── AI/
│   ├── BehaviorTrees/         — BT_Enemy_Standard, BT_Enemy_Heavy
│   ├── Blackboards/           — BB_Enemy
│   └── EQS/                   — EQS queries si las necesitás
│
├── Wave/
│   ├── WaveData/              — DA_Wave_01, DA_Wave_02, etc.
│   └── SpawnPoints/           — cualquier asset relacionado a spawn
│
├── Map/
│   ├── Geometry/              — Modular pieces del mapa (Kay assets)
│   ├── Props/                 — Props decorativos (Kay assets)
│   └── Levels/                — L_Main, L_MainMenu
│
├── VFX/
│   ├── Impacts/               — NS_BulletImpact_Concrete, NS_BulletImpact_Flesh
│   ├── Weapons/               — NS_MuzzleFlash, NS_ShellEject
│   └── Characters/            — NS_BloodSplat, NS_DeathDust
│
├── UI/
│   ├── HUD/                   — WBP_HUD, WBP_Crosshair, WBP_WaveIndicator
│   ├── Menus/                 — WBP_MainMenu, WBP_Pause, WBP_GameOver
│   └── Common/                — WBP_Button, WBP_HealthBar (widgets reutilizables)
│
├── Audio/
│   ├── SFX/                   — Organizados igual que VFX por feature
│   ├── Music/
│   └── Attenuation/           — Sound Attenuation assets compartidos
│
└── Shared/                    — Assets usados por múltiples features
    ├── Materials/             — M_Master, MI_* instances, M_PostProcess_Outline
    ├── Textures/              — T_GradientAtlas, T_LUT_Main
    ├── Decals/                — M_Decal_BulletHole, M_Decal_BloodPool
    └── Fonts/                 — Fuentes de UI
```

---

## Performance Budget (Adreno 610)

| Resource            | Budget          |
|---------------------|-----------------|
| Draw calls / frame  | < 200           |
| Triangles / frame   | < 150k          |
| RAM (total)         | < 1.5 GB        |
| Texture memory      | < 400 MB        |
| Dynamic lights      | 0–1 max         |
| Simultaneous AI     | 15–20 max       |

---

## Implementation Plan

Current phase tracking — update this section as phases complete.

| Phase | Name                        | Status      |
|-------|-----------------------------|-------------|
| F0    | Project Foundation          | ⬜ Pending  |
| F1    | Character & Camera          | ⬜ Pending  |
| F2    | Combat Base                 | ⬜ Pending  |
| F3    | Enemy AI                    | ⬜ Pending  |
| F4    | Wave System & Game Loop     | ⬜ Pending  |
| F5    | Weapons & Content           | ⬜ Pending  |
| F6    | Map & Art                   | ⬜ Pending  |
| F7    | Polish & Optimization       | ⬜ Pending  |
| FA    | Local Split-Screen (post)   | ⬜ Optional |
| FB    | Online Multiplayer (post)   | ⬜ Optional |

---

## Conventions

- Blueprint subclasses allowed for data and designer iteration, not for core logic
- All gameplay-critical code in C++ — Blueprint only for prototyping and polish
- No magic numbers — use UPROPERTY(EditDefaultsOnly) for all tunable values
- Performance-sensitive code must include a comment with the mobile budget rationale
- Commits follow conventional commits: `feat:`, `fix:`, `perf:`, `refactor:`, `docs:`

---

## What Claude Should Know When Helping

- Suggest C++ implementations first, Blueprint as fallback or for iteration
- Always consider the Adreno 610 budget before suggesting any rendering feature
- Do not suggest Lumen, Nanite, VSM, ray tracing, or any UE5 next-gen features
- When suggesting AI behavior, assume Behavior Tree + Blackboard architecture
- Kay Lousberg assets are CC0 and use Rig_Medium — no retargeting needed
- The three scalability tiers (Mobile/Switch2/PC) must always be considered
- Prefer instanced rendering (InstancedStaticMeshComponent) for repeated props
- Switch 2 support is a hard requirement — do not suggest Godot or engine changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ramo1606)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ramo1606)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
