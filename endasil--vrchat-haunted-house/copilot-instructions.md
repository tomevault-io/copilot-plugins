## vrchat-haunted-house

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A VRChat haunted house stealth game built in Unity with UdonSharp. Players collect colored pills (the keys), avoid ghost AIs, and reach the exit. All gameplay scripts are in `Assets/_3DStealthGame/Scripts/`.

There are no CLI build, lint, or test commands — compilation happens inside the Unity Editor automatically when scripts are saved. UdonSharp scripts are compiled by UdonSharp on domain reload.

## VRChat / UdonSharp Constraints

These are hard limitations imposed by the Udon VM — violating them produces compiler errors or silent failures at runtime:

- **No user-defined static classes or static methods.** Use instance methods instead. (A past comment in `LightFlickerU.cs` explicitly notes: "Removed static here since VR Chat does not support it.")
- **No `System.Linq`, `System.Threading.Tasks`, or other unsupported .NET APIs** inside UdonSharp behaviour files. These are fine in editor-only (`#if UNITY_EDITOR`) code.
- **No generic collections** (`List<T>`, `Dictionary<K,V>`). Use fixed-size arrays.
- **Deferred calls** use `SendCustomEventDelayedFrames(nameof(MyMethod), n)` — not coroutines.
- **Network authority**: AI logic that must be consistent across clients runs only on the owner: `if (!Networking.IsOwner(gameObject)) return;`
- **Player-attached scripts** (like `PlayerInventory`) are retrieved via `player.GetPlayerObjects()[0].GetComponent<T>()`, not `FindObjectOfType`.
- **NavMeshAgent** is supported from VRChat Worlds SDK ≥ 3.7.4 (this project uses 3.10.1).

## Architecture

### Inheritance / Reset System

`Resettable` (→ `UdonSharpBehaviour`) is the base class for any object that needs to be reset between attempts. On `Start()` it self-registers with the `ResetManager` singleton; subclasses override `public virtual void ResetState()`, and `ResetManager.ResetAll()` calls it on each registered behaviour. `GameEndingU` calls `ResetAll()` when the caught-screen fade finishes (per-client — everything resettable is sync mode None).

Concrete resettables: `PlayerInventory`, `PlayerUI`, `DoorU`, `Pill`.

### Ghost AIs

`GhostAgentBase` holds what the ghosts share: owner-only `NavMeshAgent` handling (with an `OnBecameAgentOwner` hook for ownership handoffs), player scanning (`FindClosestVisiblePlayer` = distance + vision cone + line-of-sight raycast; `FindClosestPlayerWithin` = plain radius for hearing), `SetDestinationOnNavMesh` (snaps points onto the mesh first), turn helpers, and the indicator text. It is never attached directly, so it has no UdonSharpProgramAsset (same as `Resettable`).

- **`GhostAISearching`** — the seeker. Patrols to random NavMesh positions, chases when a player is seen or heard, investigates the last known position after losing them, then resumes patrol with an idle look-around sweep at each stop. Receives `[NetworkCallable] OnSnowballHit(Vector3)` from `Snowball` (sent to the ghost's owner) and investigates the thrower's position.
- **`WaypointPatrolU`** — the butler. Walks a fixed `WaypointNetwork` loop with a narrow forward cone (no hearing); on sight it locks on, then marches at the player; after losing sight it returns to its route. The patrol index is `[UdonSynced]`.
- **`GargoyleU`** — marker script only (used by `ObserverU` and the editor map generator).

AI logic runs exclusively on the owner client. Catches are local: each ghost carries an `ObserverU` trigger that raycasts at the local player (a hit with `collider == null` is the player — VRChat hides player colliders) and calls `GameEndingU.CaughtPlayerBy(ghostType)`, which shows the caught overlay, broadcasts a death-feed line via a `[NetworkCallable]` event, teleports the player to spawn and triggers the reset.

### Pill / Door System

`PillColor` enum (in `Assets._3DStealthGame.Scripts` namespace, with a `LastEnum` count sentinel) drives a typed inventory stored as an enum-indexed array. `Pill` adds itself via `PlayerInventory.AddPill(pillColor)` on trigger enter and deactivates. `DoorU` checks `PlayerInventory.HasPill(pillColor)` on trigger enter and animates open (pills are not consumed).

### Player

`PlayerInventory` (extends `Resettable`) lives on the player avatar's first player object and exposes `AddPill`, `HasPill`, `GetPillCount`. `PlayerUI` owns the head-locked HUD canvas: one icon per pill color (`pillIcons`, indexed by `(int)PillColor`); it polls via `SendCustomEventDelayedFrames` until it locates `PlayerInventory` on the local player object, and on desktop re-fits the pill row and the full-screen caught/end panels to the camera every `LateUpdate`.

### Namespaces

- Most game scripts: global namespace.
- `LightFlickerU` + `FlickerMode` enum: `StealthGame` namespace (also contains editor code under `#if UNITY_EDITOR`).
- `DoorU`, `Resettable`, `PillColor`: `Assets._3DStealthGame.Scripts` namespace.
- `AwarenessIndicator`: `Assets._3DStealthGame.Scripts.Enums` namespace.

### VPM Dependencies

- `com.vrchat.worlds` 3.10.1 (VRChat SDK)
- `com.mmmaellon.smartobjectsync` 3.10.16
- `bobystarvrc.opennid` 1.0.0

---
> Source: [endasil/vrchat-haunted-house](https://github.com/endasil/vrchat-haunted-house) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
