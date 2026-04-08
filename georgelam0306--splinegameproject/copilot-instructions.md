## splinegameproject

> Scope: Root of the repo — applies to all files unless a more deeply nested AGENTS.md overrides it.

# Agent Guidelines for This Repository

Scope: Root of the repo — applies to all files unless a more deeply nested AGENTS.md overrides it.

Purpose: Keep the codebase clean, explicit, predictable, and allocation-free on hot paths in DerpLib Engine.

## Assets vs Resources (Repo Convention)

Going forward:

- `Assets/` is for **hand-authored source** content (png/gltf/json authoring, etc).
- `Resources/` is for **compiled/runtime-ready outputs** (baked binaries, compiled content pipeline outputs).
- Prefer treating `Resources/` as safe-to-regenerate output (do not hand-edit).

## Zero-Allocation Hot Path Rules

This is a real-time game where GC pauses cause visible stuttering. The per-frame loop must be allocation-free.

### Never allocate in per-frame or per-entity code

- **No lambdas/closures** — Every `() => ...` that captures variables allocates a closure object. Pass data as struct fields instead.
- **No LINQ** — `Where()`, `Select()`, `ToList()`, etc. allocate iterators and collections. Use index-based `for` loops.
- **No string operations** — `string.Format()`, interpolation `$""`, concatenation `+` all allocate. Cache strings or use `Span<char>`.
- **No boxing** — Passing a struct to a method expecting `object` or an interface allocates. Use generic constraints or concrete types.
- **No `new` for reference types** — Creating `new List<T>()`, `new Dictionary<>()`, `new Action()` in loops allocates. Reuse pooled/cached instances.
- **No delegates in hot paths** — `Action`, `Func<>`, comparison delegates all allocate unless cached as static fields.

### Preferred patterns

- Store draw parameters as struct fields, execute later with direct method calls (not delegates)
- Use `List<T>.Clear()` + reuse rather than `new List<T>()` each frame
- Cache `Comparison<T>` delegates as `static readonly` fields
- Use `ref` returns to update component data in place without copying
- Precompute atlas indices, source rectangles, and lookup tables at load time
- Keep hot path data in contiguous arrays for cache-friendly iteration

### Avoid dictionaries on hot paths

Dictionaries have overhead that is suboptimal for per-frame lookups:
- Hash computation on every access
- Cache-unfriendly memory layout (pointer chasing)
- Bucket collision handling

**Prefer fixed-size arrays when bounds are known:**

Bad (dictionary with tuple key):
```csharp
private Dictionary<(int, int), FlowCell[]> _chunks;
var cells = _chunks[(chunkX, chunkY)]; // hash computation every lookup
```

Good (bounded 2D array with coordinate offset):
```csharp
private FlowCell[][] _chunkGrid; // 3x3 grid centered on player
int localX = chunkX - _centerChunkX + 1;
int localY = chunkY - _centerChunkY + 1;
var cells = _chunkGrid[localX * 3 + localY]; // direct index, O(1)
```

**When dictionaries are acceptable:**
- One-time initialization lookups (not per-frame)
- Truly unbounded/sparse data with unpredictable keys
- Editor/tooling code outside the game loop

### Example: Allocation-free render queue

Bad (allocates every call):
```csharp
RenderQueue.Add(sortY, () => Derp.DrawTexturePro(texture, src, dst, originX, originY, rotation, r, g, b, a));
```

Good (zero allocations):
```csharp
RenderQueue.Add(sortY, texture, sourceRect, destRect, origin, rotation, tint);
// Flush system calls Derp.DrawTexturePro with stored struct data
```

## Core Rules

### No runtime fallbacks
- Do not introduce alternate code paths like "try this API, otherwise use that" or conditional calls for "older builds".
- Prefer explicit, compile-time decisions (e.g., `#if WINDOWS` / `#if LINUX` / per-target files) over runtime checks and fallbacks.
- If a required API is unavailable, fail the build or surface a clear error rather than silently switching behavior.

### Platform separation
- Use preprocessor symbols and/or separate files/projects for platform-specific logic (e.g., client vs. headless tools).
- Avoid broad `try/catch` blocks that swallow platform errors. Make incompatibilities explicit.

### Engine boundary (DerpLib)
- All rendering/input should go through DerpLib Engine APIs (e.g., `Derp.*`) from game code.
- Keep backend/windowing/Vulkan-specific code confined to the Engine layer; do not duplicate engine primitives in game projects.

### Trimming and native bindings
- For native or P/Invoke wrappers used by DerpLib Engine (e.g., graphics/windowing bindings), preserve required APIs via a linker descriptor or disable trimming per target.
- Do not add runtime substitutions for missing methods. Fix the build/link configuration instead.

### Dependency Injection
- Keep a single composition root and a single intended wiring. Do not add optional/alternative DI graphs unless explicitly requested.

## DerpDoc MCP Workflow Rules

- Always resolve and use the active project before any MCP table read or mutation.
- For multi-row or multi-cell operations, use MCP batch APIs by default.
- Do not loop single-row MCP calls for bulk edits unless no batch API exists.

## DerpDoc Codegen Rules

- Gameplay/runtime code must consume DerpDoc game data through generated APIs (`GameDatabase`, generated row structs, generated table accessors).
- `Database/tables/*.json*` is authoring-only input. Do not read it at runtime in gameplay code.
- Do not create hand-written DTO mirrors of exported tables in gameplay/runtime code.
- If DerpDoc export/codegen fails, stop and surface the export/codegen error clearly; do not introduce fallback JSON loaders.
- If required runtime data is missing, fix the DerpDoc schema/export/codegen path rather than shadowing it in game code.
- Exception: tooling/editor/debug utilities that intentionally operate on authoring files; these must stay outside runtime gameplay paths.

## Code Style

- Keep implementations minimal and targeted; avoid compatibility layers.
- No dead code, no commented-out alternatives, no "just in case" branches.
- Document non-obvious platform constraints in code comments near the implementation, not as runtime fallbacks.
- One top-level type per file (class/struct/record). Do not place multiple classes in a single file. Use partials to split a single type across files if needed.
- Use descriptive variable names. Never use single-letter names (except `i`, `j`, `k` for loop iteration). Avoid abbreviations like `sw`, `sh`, `tf`, `sr` — use `srcTileWidth`, `srcTileHeight`, `transform`, `spriteRenderer` instead. Names should be self-documenting.
- Always use braces for control flow (`if`, `for`, `while`, `foreach`), even for single statements.
- Never inline multiple statements on one line.

## Friflo ECS Guidelines

### No structural changes inside query loops

Friflo ECS throws `StructuralChangeException` if you add/remove components or delete entities while iterating a query. This is a common source of runtime crashes.

**Never do this inside `foreach (var entity in Query.Entities)` or similar:**
- `entity.AddComponent<T>()`
- `entity.RemoveComponent<T>()`
- `store.CreateEntity()`
- Direct entity deletion

**Always use CommandBuffer for deferred operations:**

Bad (crashes at runtime):
```csharp
foreach (var entity in Query.Entities)
{
    if (shouldDie)
    {
        entity.AddComponent<MarkedForDeath>(); // CRASH: StructuralChangeException
    }
}
```

Good (deferred execution):
```csharp
foreach (var entity in Query.Entities)
{
    if (shouldDie)
    {
        CommandBuffer.AddComponent<MarkedForDeath>(entity.Id);
    }
}
CommandBuffer.Playback(); // Apply all changes after loop
```

**Alternative patterns:**
- Collect entity IDs in a list, process after the loop
- Use a service to queue operations for later (like `WorldMapService.MarkChunkDirty()`)
- For complex multi-query operations, use `CommandBuffer` from the system base class

## DerpLib Engine Rendering Guidelines

- Call `Derp.PollEvents()` and `Derp.BeginDrawing()` once per frame, not per entity/system.
- Avoid redundant render state changes inside inner loops; rely on engine batching instead of per-entity draw setup.
- For pixel art content, configure textures/samplers for point/nearest filtering at load time (not during rendering).

## Determinism Requirements

This game requires deterministic simulation for rollback netcode and replay systems. All simulation code must produce identical results given identical inputs.

### Always use Fixed64 for simulation math

- Never use `float` or `double` in simulation logic — floating-point results vary across platforms and builds.
- Use `Fixed64` (48.16 fixed-point) for all positions, velocities, distances, and calculations in the simulation layer.
- `float` is acceptable only in rendering code that does not affect game state.

### Deterministic iteration order

- Never iterate over `HashSet<T>` or `Dictionary<K,V>` in simulation code — iteration order is undefined.
- Use `List<T>` with explicit sorting, or arrays with fixed indices.
- When processing entities, always iterate in a deterministic order (e.g., entity ID).

### No randomness without seeded RNG

- Never use `System.Random` without a fixed seed.
- Use a deterministic RNG seeded from game state for all simulation randomness.

### Run tests before committing simulation changes

After modifying any simulation code, always run:
```bash
dotnet test DerpTech2D.Tests/DerpTech2D.Tests.csproj
```

All determinism tests must pass. If a test fails, the change breaks determinism and must be fixed before committing.

## Review Checklist

- Does this change allocate in a per-frame or per-entity loop? If yes, refactor to be allocation-free.
- Are any lambdas/closures used in hot paths? If yes, replace with struct data + direct calls.
- Does this change introduce any runtime fallback or alternate execution path? If yes, remove it.
- Is platform-specific logic resolved at compile time? If not, refactor.
- Are required native bindings preserved by trimming/linker configuration rather than by code fallbacks?
- Does game code consistently go through DerpLib Engine APIs rather than backend-specific calls?
- Are pixel art textures configured for point/nearest filtering at load time?
- Are dictionaries used in hot paths? If yes, replace with bounded arrays when key ranges are predictable.
- Are there any structural ECS changes (AddComponent, RemoveComponent, CreateEntity, DeleteEntity) inside query loops? If yes, use CommandBuffer instead.
- Does simulation code use `float`/`double`? If yes, replace with `Fixed64`.
- Does simulation code iterate over `HashSet` or `Dictionary`? If yes, use deterministic collections.
- Are exported DerpDoc tables consumed through generated code (not hand-written DTO mirrors or direct table JSON parsing)?
- Do all determinism tests pass after this change?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/georgelam0306)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/georgelam0306)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
