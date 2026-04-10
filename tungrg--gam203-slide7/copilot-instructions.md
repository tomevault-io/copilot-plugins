## gam203-slide7

> - **Engine**: Unity, Universal Render Pipeline (URP 17.3.0)

# GitHub Copilot Instructions — Unity 2D Game

## Project Setup

- **Engine**: Unity, Universal Render Pipeline (URP 17.3.0)
- **Type**: 2D game (2D Renderer, Sprite-based)
- **Language**: C# (Unity scripting API)
- **Input**: New Input System (`com.unity.inputsystem` 1.17.0) — NEVER use legacy `UnityEngine.Input`

## Packages in Use

- `com.unity.2d.animation` 13.0.2
- `com.unity.2d.aseprite` 3.0.1
- `com.unity.2d.psdimporter` 12.0.1
- `com.unity.2d.sprite` 1.0.0
- `com.unity.2d.spriteshape` 13.0.0
- `com.unity.2d.tilemap` 1.0.0 + `com.unity.2d.tilemap.extras` 6.0.1
- `com.unity.inputsystem` 1.17.0
- `com.unity.render-pipelines.universal` 17.3.0
- `com.unity.timeline` 1.8.9
- `com.unity.ugui` 2.0.0
- `com.unity.visualscripting` 1.9.9

## C# Coding Rules

### Naming Conventions
- Types and methods: `PascalCase`
- Private fields: `_camelCase`
- Serialized Inspector fields: `[SerializeField] private type fieldName;` (`camelCase`, no underscore)
- Constants: `UPPER_SNAKE_CASE`

### MonoBehaviour Structure
```csharp
// 1. Serialized fields
[SerializeField] private float moveSpeed = 5f;

// 2. Private fields
private Rigidbody2D _rb;
private PlayerInput _input;

// 3. Awake — cache components, self-init
private void Awake()
{
    _rb = GetComponent<Rigidbody2D>();
    _input = GetComponent<PlayerInput>();
}

// 4. OnEnable — subscribe to events/input
private void OnEnable() { }

// 5. Start — cross-object references
private void Start() { }

// 6. Update / FixedUpdate
private void FixedUpdate() { }

// 7. OnDisable — unsubscribe
private void OnDisable() { }

// 8. OnDestroy — cleanup
private void OnDestroy() { }
```

### Hard Rules
- `[SerializeField] private` — never `public` fields
- Cache `GetComponent<>()` in `Awake()` — never in `Update()`
- Never use `GameObject.Find()` at runtime
- Never use `Resources.Load()` — use direct references
- Never use `Camera.main` in loops — cache it
- Always unsubscribe events in `OnDisable()` / `OnDestroy()`
- Physics/movement code goes in `FixedUpdate()`
- Use `Rigidbody2D.interpolation = Interpolate` for player movement smoothness

## Input System
- Use `InputAction` callbacks or the generated C# class from `Assets/InputSystem_Actions.inputactions`
- Subscribe in `OnEnable()`, unsubscribe in `OnDisable()`
```csharp
private void OnEnable() => _moveAction.performed += OnMove;
private void OnDisable() => _moveAction.performed -= OnMove;
```

## Tilemap
- Use **Rule Tiles** from `com.unity.2d.tilemap.extras` — not manual tile-by-tile placement
- Use `CompositeCollider2D` on tilemap colliders

## Lighting & Rendering
- Use `Light2D` (URP 2D) — not `Light`
- Use URP Sprite shaders (`Sprite/Lit`, `Sprite/Unlit`)
- Control draw order with Sorting Layers

## Architecture Patterns
- **Singleton Managers**: `GameManager`, `AudioManager`, `SceneLoader` — keep them thin
- **ScriptableObjects**: all shared data (stats, configs, event channels)
- **Object Pooling**: bullets, enemies, VFX — never instantiate/destroy in tight loops
- **State Machines**: player, enemy, and game state logic
- **Events**: C# `event Action` or ScriptableObject event channels for decoupling

## Folder Structure
```
Assets/_Game/Scripts/
  Characters/
  Core/
  Enemies/
  Input/
  Managers/
  UI/
  Utilities/
```

## Testing

- After every **function**, generate a **NUnit `[Test]`** (EditMode) for it.
- After every **feature** (scene/runtime required), generate a **`[UnityTest]`** returning `IEnumerator` (PlayMode) for it.
- When generating both code and tests, always generate the **Assembly Definition (`.asmdef`) files** below.

### Assembly Definitions

**Runtime** — `Assets/_Game/Scripts/<Domain>/<Domain>.asmdef`:
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

**Tests** — `Assets/_Game/Tests/<Domain>/<Domain>.Tests.asmdef`:
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

### EditMode test (`[Test]`)
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

### PlayMode test (`[UnityTest]`)
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
        var go = new GameObject();
        var component = go.AddComponent<MyComponent>();
        yield return null;
        Assert.IsTrue(component.IsReady);
        Object.Destroy(go);
    }
}
```

## What to Avoid
- Legacy `UnityEngine.Input` class
- `GetComponent<>()` in `Update()`
- `GameObject.Find()`, `FindObjectOfType()` at runtime
- `Resources.Load()`
- `WWW`, `OnGUI`, `GUI.*`, `GUILayout`
- `public` fields on MonoBehaviours for Inspector access
- Forgetting event unsubscription (causes memory leaks and ghost callbacks)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tungrg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
