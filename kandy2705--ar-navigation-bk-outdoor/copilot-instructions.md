## ar-navigation-bk-outdoor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AR outdoor navigation app for Bach Khoa (BK) university campus, built with Unity 2022.3.50f1. The app uses GPS and AR anchors to guide users across the campus by rendering a navigation path in AR.

## Build & Development

This is a Unity project — there are no CLI build commands. All development is done inside the **Unity Editor (2022.3.50f1)**:

- Open the project in Unity Hub using Unity 2022.3.50f1
- Target platform: **Android** (configured in `Assets/Plugins/Android/`)
- Build via **File > Build Settings > Android > Build**
- To test in Editor without a device: most scripts have `#if UNITY_EDITOR` blocks that simulate AR behavior (e.g., `NewIndoorNav.SimulateImageDetected()` and `GPSMarker.useMockMovement`)

### Android Requirements

- `AndroidManifest.xml` declares `INTERNET` and `ACCESS_NETWORK_STATE` permissions
- GPS and AR features require a physical Android device with ARCore support
- The app targets outdoor use; GPS permission must be granted at runtime

## Architecture

The project has two distinct navigation modes:

### 1. Outdoor GPS Navigation (`Assets/Code/`)

Coordinate flow: **GPS → ECEF → ENU → Unity local space**

- **`GPSMarker.cs`** — Core GPS component. Converts WGS84 lat/lon to ECEF, then to local ENU coordinates relative to a reference point (`refLat`/`refLon` set to BK campus origin). Positions the user marker on the `mapPlane`. Has mock movement mode (`useMockMovement`) and mock compass (`useMockCompass`) for editor testing.
- **`NavigationTarget.cs`** — Positions a destination object on the `mapPlane` using the same ECEF→ENU conversion from `GPSMarker`.
- **`AlignXROriginToUser.cs`** — One-shot alignment: shifts the `XROrigin` so the AR camera matches the GPS-calculated user position.
- **`SetNavigation.cs`** — Renders the navigation path as a flat mesh strip using Unity `NavMesh.CalculatePath()`. Samples both user and target to the NavMesh before pathfinding. Path visibility is gated by `GlobalProperties.Instance.IsShowNavigation`.
- **`GeoAnchorController.cs`** — Wraps ARCore Extensions `ARGeospatialAnchor` creation; waits for `EarthTrackingState == Tracking` before placing geo-anchors.
- **`GlobalProperties.cs`** — Singleton (DontDestroyOnLoad) holding shared state; currently only `IsShowNavigation` flag.

### 2. Indoor Image-Tracking Navigation (`Assets/Code/NewIndoorNav.cs`)

- Detects a physical marker image via `ARTrackedImageManager`
- On detection, instantiates a `trackedImagePrefab` containing `NavigationTarget` children and a `NavMeshSurface`
- Builds the NavMesh at runtime after spawning, then renders the path via a `LineRenderer`
- In Editor, `SimulateImageDetected()` spawns the prefab at a fake position for testing

### POI System

- **`POI.cs`** — Extends `ListItemData`; holds name, description, type (`POIType` enum), and references to `POISign` and `POICollider`
- **`POISign.cs`** — Billboard sign; uses partial class pattern to handle clicks differently on desktop (`IPointerClickHandler`) vs mobile (`IPointerDownHandler`). Click blocked beyond 7.5m.
- **`POICollider.cs`** / **`DescriptionCollider.cs`** — Trigger-based arrival detection

### Key Dependencies (Packages)

| Package | Version | Purpose |
|---|---|---|
| `com.google.ar.core.arfoundation.extensions` | git | ARCore Geospatial API |
| `com.immersal.core` | git | Immersal visual positioning |
| `com.unity.ai.navigation` | 1.1.7 | NavMesh baking at runtime |
| `com.unity.xr.arcore` / `arkit` | 5.1.6 | AR Foundation |

## Key Coordinates

The reference origin for ENU calculations is hardcoded in `GPSMarker.cs`:
- `refLat = 10.7736444`, `refLon = 106.6593743` (BK campus, Ho Chi Minh City)
- Default user spawn: `lat = 10.7741875`, `lon = 106.6606904`

## Scene Structure Notes

- The `mapPlane` Transform is the parent that holds all GPS-positioned objects; rotating/moving it adjusts the entire map
- `XROrigin` is aligned to match the GPS user position exactly once on startup via `AlignXROriginToUser`
- `SetNavigation` requires a NavMesh to be baked in the scene for outdoor paths
- The `InputManager.cs` file contains `ChatInputLegacy` — a keyword-based chat UI that sets `IsShowNavigation = true` when building names (B10, B9, B8, A4, B4) are mentioned

---
> Source: [Kandy2705/AR-Navigation-BK-Outdoor](https://github.com/Kandy2705/AR-Navigation-BK-Outdoor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
