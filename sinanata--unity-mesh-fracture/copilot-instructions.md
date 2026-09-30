## unity-mesh-fracture

> Guidance for AI coding agents (Codex, Cursor, GitHub Copilot, Claude Code, Windsurf, Aider, Zed, and others) working in this repository or wiring this tool into another project. Humans: this is a fast, accurate map. Deeper docs are linked at the bottom.

# AGENTS.md

Guidance for AI coding agents (Codex, Cursor, GitHub Copilot, Claude Code, Windsurf, Aider, Zed, and others) working in this repository or wiring this tool into another project. Humans: this is a fast, accurate map. Deeper docs are linked at the bottom.

## What this project is

A drop-in **Voronoi mesh fracturer for Unity 6 + URP**. Pure-C# fragmentation (zero dependencies beyond `UnityEngine`) that pre-bakes watertight two-submesh fragments — with cooked convex hulls — at load time, then plays them as a fade-out debris burst with optional Unity physics. The entire runtime is three files under `Assets/MeshFracture/Runtime/`. Namespace: `MeshFracture`. License: MIT. Battle-tested in the shipping cross-platform game Leap of Legends, where every character that explodes runs through this pipeline.

## Golden rules (do not violate)

1. **"Burst" here is the explosion, not Unity's Burst compiler.** `FractureBurst` is a plain `MonoBehaviour`; the whole runtime is single-threaded managed C# over `UnityEngine` types (`Mesh`, `Vector3`, `List<T>`, `Quaternion`). Do **not** add `[BurstCompile]`, `IJob*`, `NativeArray<T>`, `Unity.Mathematics.float3`, or `using Unity.Collections / Unity.Burst / Unity.Jobs`. None are dependencies; none are used. Pattern-matching "Burst mesh fracture" onto the DOTS/Jobs stack is the single most common wrong turn here.
2. **Everything runs on the main thread; bake at load, look up at impact.** `MeshFragmenter.Fragment` is a synchronous, O(n²)-in-fragment-count CPU call. Never call it per-impact during gameplay. Use `FragmentCache.RequestPreBake` (async, one bake per frame) or `FragmentCache.BakeSynchronous` (blocking, loading-screen only) at load time, then `FragmentCache.TryGet` at the moment of impact. `TryGet` returning `false` means "not baked yet" — fall back to a particle/smoke puff, don't block.
3. **Source meshes must be Read/Write-enabled.** `Fragment` reads `source.vertices / normals / uv / triangles`; on a non-readable imported mesh those come back empty and you get a single-fragment fallback. Enable Read/Write on the model importer (the demo's `DemoModelImporter` does this for its FBXes).
4. **Fragment materials must be transparent and owned per-burst.** `Initialize` animates `_BaseColor.a` 1 → 0 by mutating the material instance directly (MaterialPropertyBlock overrides are silently dropped on some URP / SRP-Batcher / WebGL combinations). Hand each burst its own `Material` instances (`new Material(interiorMat)`), configured transparent + Cull Off + **ZWrite ON** — copy `MaterialFactory.ConfigureFractureTransparent`. ZWrite must stay ON, or solid chunks render see-through-to-interior.
5. **Two submeshes per fragment:** submesh 0 = exterior (the original textured surface), submesh 1 = cap/interior (raw cut faces). Always pass both an exterior and an interior material to `Initialize`.
6. **No custom `.shader` assets.** The runtime ships zero `.shader` and zero `.mat` files on purpose — a hand-rolled URP shader rendered every fragment solid black on WebGL2 (an HLSL → GLSL ES 3.0 cross-compile bug). Stock URP/Lit configured at runtime sidesteps it. Do not reintroduce a custom shader.
7. **No `using LeapOfLegends.*` or other product-specific imports.** New runtime code lives in the `MeshFracture` namespace only; demo-only code lives under `MeshFractureDemo`.

## Wire it into your project

The runtime is one folder — `Assets/MeshFracture/` — dropped into your `Assets/`. The full gameplay wiring is:

```csharp
using MeshFracture;

// 1. At load (a loading screen), pre-bake. Voronoi is ~3-9 ms/character on desktop.
int key = prefab.GetInstanceID() ^ (fragmentCount * 7919);
FragmentCache.RequestPreBake(key, prefab, fragmentCount);

// 2. At impact, look up and spawn the burst.
if (!FragmentCache.TryGet(key, out var cached)) return;   // not baked yet -> particle fallback
Vector3 offset = deathPos - cached.MeshCenter;
var fragments = new FragmentResult[cached.Meshes.Length];
for (int i = 0; i < fragments.Length; i++)
    fragments[i] = new FragmentResult { Mesh = cached.Meshes[i], Centroid = cached.LocalCentroids[i] + offset };

var burst = new GameObject("FractureBurst").AddComponent<FractureBurst>();
burst.transform.position = deathPos;
burst.Initialize(fragments, exteriorMat, new Material(interiorMat),
                 explosionForce: 18f, meshRotation: cached.MeshRotation);
```

For bouncing, collision-aware chunks, set these before `Initialize`: `burst.UseUnityPhysics = true; burst.PreBakedColliderMeshes = cached.ColliderMeshes; burst.DissolveAfterSettle = true;`. The canonical minimal example is `Assets/MeshFracture/Demo/MeshFractureDemo.cs` (~140 lines, drop-on-a-cube).

## Public API

`MeshFragmenter` (static — `Assets/MeshFracture/Runtime/MeshFragmenter.cs`):
```csharp
static FragmentResult[] Fragment(Mesh source, int count, Vector3 center);   // deterministic from (source, center)
static Mesh BakeSkinnedMesh(SkinnedMeshRenderer smr);                       // SMR-local space; BakeMesh(useScale:false) — do not double-scale
struct FragmentResult { Mesh Mesh; Vector3 Centroid; }
```
`FragmentCache` (static — `FragmentCache.cs`):
```csharp
static void RequestPreBake(int key, GameObject modelPrefab, int fragmentCount);   // async, one bake/frame
static void BakeSynchronous(int key, Mesh sourceMesh, Quaternion meshRotation, int fragmentCount);
static bool TryGet(int key, out CachedData data);                                 // O(1) impact-time lookup
static Mesh BuildAndBakeColliderMesh(Mesh source);
static void Evict(int key);  static void Clear();
struct CachedData { Mesh[] Meshes; Vector3[] LocalCentroids; Vector3 MeshCenter; Quaternion MeshRotation; Mesh[] ColliderMeshes; }
```
`FractureBurst : MonoBehaviour` (`FractureBurst.cs`):
```csharp
void Initialize(FragmentResult[] fragments, Material exteriorMaterial, Material interiorMaterial,
                float explosionForce = 18f, float trailWidth = 0.12f,
                Gradient trailGradient = null, Quaternion meshRotation = default);
// Tunables (set before Initialize): Lifetime, DissolveDuration, DissolveAfterSettle, SettleHoldDuration,
// UseUnityPhysics, Bounciness, Friction, FragmentMass, GravityVector, LockToXY, LockPlaneNormal,
// EnableTrails, PreBakedColliderMeshes.   Read-only: IsFullyInitialized.
```
CPU sim is the default (`GravityVector`, a cheap `y = -1` floor). `UseUnityPhysics = true` swaps in a `Rigidbody` + convex `MeshCollider` per fragment and uses `Physics.gravity`; pass `PreBakedColliderMeshes = cached.ColliderMeshes` to skip the per-spawn hull cook. `LockToXY` / `LockPlaneNormal` are CPU-sim-only (2D / sprite bursts — physics mode has no arbitrary-plane constraint).

## Repository layout

- `Assets/MeshFracture/` — **the shippable tool, and the only folder a consumer copies.** `Runtime/` (MeshFragmenter, FragmentCache, FractureBurst) + `Demo/MeshFractureDemo.cs` (the minimal example).
- `Assets/Demo/` — the WebGL showcase host project (eight pedestals, a UI Toolkit overlay, sprite destructibles). Namespace `MeshFractureDemo`. Not part of the tool: do not copy it into a consuming project, and do not reference it from runtime code.
- `Assets/Editor/`, `Assets/Settings/`, `Assets/WebGLTemplates/` — build + URP scaffolding for the demo.
- `Tools/` — the shared build orchestrator (`Tools/.orchestrator` submodule) plus a thin `Build-Demo.ps1` shim.
- `Vendor/` — the design-system and sprite-baker submodules, used only by the demo.

## Conventions when editing

- **Comments answer "why", not "what".** Explain why 0.8 and not 0.5 — the trade-off, not the line.
- **Watertightness is load-bearing.** `MeshFragmenter` keeps one triangle list with per-triangle submesh tags and re-clips caps against every plane; the dedup-cut-edges-by-position step is what keeps concave / hollow meshes from showing through. Read the comments before touching the clip / cap math.
- **`BakeSkinnedMesh` must not double-apply scale.** It calls `smr.BakeMesh(baked, useScale: false)` because the no-arg overload already multiplies by lossy scale on Unity 2021+; a caller that also pre-scales gets 1.69×-too-big fragments on a 1.3× character.
- **Determinism:** `Fragment` is deterministic from (source mesh, center) within a run — same inputs, same output frame after frame.

## Build, preview, validate

Windows-first Unity 6 project (host editor 6000.3.8f1). There is no unit-test suite; validation is visual, through the two demos.

- **Editor preview:** open `Assets/Demo/Scenes/MeshFractureDemo.unity` and press Play. The minimal example is `Assets/MeshFracture/Demo/MeshFractureDemo.cs` — drop it on a Cube with a MeshFilter/MeshRenderer and press Space.
- **WebGL build (what visitors see), from the repo root in PowerShell:**
  ```powershell
  git submodule update --init --recursive           # first time: the orchestrator is a submodule
  copy Tools\Build\config.example.json Tools\Build\config.local.json
  .\Tools\Build\Build-Demo.ps1 -Serve               # builds to build/WebGL/ and serves http://localhost:3000
  ```
  `-Serve` runs a local server, `-Deploy` force-pushes a single commit to `gh-pages`, `-ClearCache` recovers from a stale Burst-AOT cache.
- **Verify both demos** after any API change (drop-on-a-cube + the WebGL scene), and confirm the WebGL build matches the editor — that is what catches the WebGL2 shader-black regression.

## Pull request checklist (summary; full list in CONTRIBUTING.md)

- Simple demo still works (drop on a Cube, Space, fragments fall → fade → gone at `Lifetime`).
- WebGL demo still builds + works (`Tools\Build\Build-Demo.ps1 -Serve`).
- No `using LeapOfLegends.*` or product-specific imports.
- Comments answer why, not what.
- If you touched the transparent material setup: **ZWrite stays ON**.
- No new `.shader` assets (the WebGL2 black-fragment bug lives there).
- `CHANGELOG.md` updated; README updated if the public API or behaviour changed.

## Deeper docs

- Full walkthrough, architecture, and the material recipe: `README.md`
- Contribution rules and PR checklist: `CONTRIBUTING.md`
- Version history: `CHANGELOG.md`
- Machine-readable index: `llms.txt`
- Sibling tools: [design system](https://github.com/sinanata/unity-ui-toolkit-design-system) · [3D-to-sprite baker](https://github.com/sinanata/unity-3d-to-sprite-baker) · [prefab-thumbnail renderer](https://github.com/sinanata/unity-prefab-thumbnail-renderer) · [build orchestrator](https://github.com/sinanata/unity-cross-platform-local-build-orchestrator)

---
> Source: [sinanata/unity-mesh-fracture](https://github.com/sinanata/unity-mesh-fracture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
