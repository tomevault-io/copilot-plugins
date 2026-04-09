## gam203-slide7

> - **Engine**: Unity (Universal Render Pipeline 17.3.0)

# Unity 2D Project — Claude Instructions

## Project Overview

- **Engine**: Unity (Universal Render Pipeline 17.3.0)
- **Type**: 2D game
- **Language**: C# (.NET, Unity scripting API)
- **Input**: Unity Input System 1.17.0 (new system — do NOT use legacy `Input` class)

## Key Packages

| Package | Version | Purpose |
|---|---|---|
| com.unity.2d.animation | 13.0.2 | Skeletal 2D animation |
| com.unity.2d.aseprite | 3.0.1 | Aseprite sprite import |
| com.unity.2d.psdimporter | 12.0.1 | PSD file import |
| com.unity.2d.sprite | 1.0.0 | Core sprite tools |
| com.unity.2d.spriteshape | 13.0.0 | Freeform 2D terrain/shapes |
| com.unity.2d.tilemap | 1.0.0 | Tilemap system |
| com.unity.2d.tilemap.extras | 6.0.1 | Rule Tiles, Animated Tiles |
| com.unity.inputsystem | 1.17.0 | New Input System |
| com.unity.render-pipelines.universal | 17.3.0 | URP rendering |
| com.unity.timeline | 1.8.9 | Cutscene / sequence timeline |
| com.unity.ugui | 2.0.0 | UI (Canvas-based) |
| com.unity.visualscripting | 1.9.9 | Visual scripting graphs |

## Code Style & Conventions

### General C# Rules
- Use `PascalCase` for class names, method names, and public fields/properties.
- Use `camelCase` for private fields. Prefix private serialized fields with `[SerializeField]` — do NOT make fields public just for Inspector access.
- Use `_camelCase` (underscore prefix) for private instance fields that are not serialized.
- Prefer `readonly` where possible.
- Do not use `var` unless the type is obvious from the right-hand side.
- Null checks: prefer `if (obj == null)` over null-conditional for Unity objects (`obj?.Method()` can behave unexpectedly with destroyed Unity objects).

### Unity-Specific Rules
- Use `[SerializeField] private` instead of `public` for Inspector-exposed fields.
- Never use `GameObject.Find` or `FindObjectOfType` at runtime in Update loops — cache references in `Awake`/`Start` or via the Inspector.
- Use `Awake()` for self-initialization and `Start()` for cross-object references.
- Prefer **ScriptableObjects** for shared configuration/data (stats, item definitions, settings).
- Use **Object Pooling** for frequently spawned/destroyed objects (enemies, bullets, particles).
- Avoid `Update()` polling when event-driven approaches work (use C# events, UnityEvents, or the new Input System callbacks).

### Input System
- Always use `InputActionAsset` or generated C# classes from the Input Actions asset.
- The project's actions file is `Assets/InputSystem_Actions.inputactions`.
- Subscribe/unsubscribe from input callbacks in `OnEnable`/`OnDisable`.

### Physics 2D
- Use `Rigidbody2D` + `Collider2D` for all physics-based objects.
- Set `Rigidbody2D.interpolation = RigidbodyInterpolation2D.Interpolate` for player-controlled characters.
- Use `Physics2D.OverlapCircle` / `OverlapBox` for lightweight overlap checks instead of triggers where appropriate.
- Prefer `FixedUpdate()` for physics/movement code.

### Rendering / URP 2D
- Use **2D Renderer** (Renderer2D.asset) — not the forward/deferred 3D renderer.
- Use **URP 2D Lights** (`Light2D`) for lighting — do not use legacy Light components.
- Sprites should use the **Sprite/Lit** or **Sprite/Unlit** URP shaders.
- Use **Sorting Layers** and **Order in Layer** intentionally; define them in Project Settings → Tags & Layers.

### Tilemap
- Use Rule Tiles (`com.unity.2d.tilemap.extras`) for terrain variation instead of hand-placing individual tiles.
- Keep each tilemap on its own dedicated GameObject with its own `TilemapCollider2D` if it needs collision.
- Use `CompositeCollider2D` on tilemap colliders for performance.

## Project Structure Conventions

```
Assets/
  _Game/               ← all project-specific assets live here
    Animations/
    Audio/
    Fonts/
    Materials/
    Physics/
    Prefabs/
      Characters/
      Enemies/
      Projectiles/
      UI/
    Scenes/
    ScriptableObjects/
    Scripts/
      Characters/
      Core/
      Enemies/
      Input/
      Managers/
      UI/
      Utilities/
    Sprites/
      Characters/
      Environment/
      UI/
    Tilemaps/
```

- Keep third-party assets in `Assets/Plugins/` or `Assets/ThirdParty/`.
- Do NOT put scripts in the root `Assets/` folder.

## Architecture Patterns

- **Manager pattern**: Use a `GameManager` singleton (via a static instance) for global game state. Keep it lean — delegate to sub-managers (AudioManager, SceneLoader, etc.).
- **Event-driven communication**: Use C# `Action`/`event` or a lightweight EventBus ScriptableObject to decouple systems.
- **ScriptableObject data**: Separate data from behaviour. Character stats, wave configs, item definitions → ScriptableObjects.
- **State machines**: For character/enemy AI, implement a simple state machine (enum-based or class-based) — avoid bloated switch statements in `Update()`.

## What Claude Should Do

- When writing new scripts, always include `using UnityEngine;` and other required namespaces.
- When creating MonoBehaviours, follow the `Awake → OnEnable → Start → Update → OnDisable → OnDestroy` lifecycle order.
- Prefer composition over inheritance for components.
- When asked to fix a bug, explain the root cause before providing the fix.
- When generating prefab or scene setup instructions, be explicit about component settings and values.
- Do not generate code that uses deprecated Unity APIs (`OnGUI`, `GUI.*`, legacy `Input`, `WWW`, etc.).
- If a task requires a Unity package not listed above, call it out explicitly before writing code that depends on it.

## Testing

### Rules
- After every **function**, generate a corresponding **NUnit `[Test]`** for it in an EditMode test file.
- After every **feature** (requires scene/runtime), generate a **`[UnityTest]`** (returns `IEnumerator`) in a PlayMode test file.
- Whenever both implementation code and tests are generated, also generate the required **Assembly Definition (`.asmdef`) files**.

### Assembly Definitions
When generating code + tests, always produce:

**Runtime assembly** (`Assets/_Game/Scripts/<Domain>/<Domain>.asmdef`):
```json
{
  "name": "Game.<Domain>",
  "references": [],
  "includePlatforms": [],
  "excludePlatforms": [],
  "allowUnsafeCode": false,
  "overrideReferences": false,
  "autoReferenced": true
}
```

**Tests assembly** (`Assets/_Game/Tests/<Domain>/<Domain>.Tests.asmdef`):
```json
{
  "name": "Game.<Domain>.Tests",
  "references": [
    "UnityEngine.TestRunner",
    "UnityEditor.TestRunner",
    "Game.<Domain>"
  ],
  "includePlatforms": [],
  "excludePlatforms": [],
  "allowUnsafeCode": false,
  "overrideReferences": true,
  "precompiledReferences": ["nunit.framework.dll"],
  "autoReferenced": false,
  "defineConstraints": ["UNITY_INCLUDE_TESTS"]
}
```

### EditMode test template (`[Test]`)
```csharp
using NUnit.Framework;

public class MyClassTests
{
    [Test]
    public void MethodName_StateUnderTest_ExpectedBehavior()
    {
        // Arrange

        // Act

        // Assert
        Assert.AreEqual(expected, actual);
    }
}
```

### PlayMode test template (`[UnityTest]`)
```csharp
using System.Collections;
using NUnit.Framework;
using UnityEngine;
using UnityEngine.TestTools;

public class MyFeatureTests
{
    [UnityTest]
    public IEnumerator FeatureName_Condition_ExpectedOutcome()
    {
        // Arrange
        var go = new GameObject();
        var component = go.AddComponent<MyComponent>();

        // Act
        yield return null; // wait one frame

        // Assert
        Assert.IsTrue(component.IsReady);

        // Cleanup
        Object.Destroy(go);
    }
}
```

## Common Pitfalls to Avoid

- Do NOT call `GetComponent<>()` in `Update()` — cache in `Awake()`.
- Do NOT destroy objects during iteration — use deferred lists.
- Do NOT use `Resources.Load` — use Addressables or direct references.
- Do NOT use `Camera.main` in tight loops — cache the camera reference.
- Do NOT mix new Input System and legacy `Input` class.
- Do NOT forget to unsubscribe from events in `OnDisable`/`OnDestroy` to prevent memory leaks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tungrg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tungrg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
