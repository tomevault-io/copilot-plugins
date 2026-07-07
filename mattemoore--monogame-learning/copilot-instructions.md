## monogame-learning

> This is a **C# MonoGame** project designed for learning game development concepts, with the primary goal of building a side-scrolling beat 'em up game in the vein of 90s classics like *Streets of Rage*, *Final Fight*, and *Double Dragon*. As part of this effort, we are developing a generic, reusable core engine library (`MonoGameLearning.Core`) that will be useful for bootstrapping future game projects.

# MonoGame Learning Project

## Project Overview

This is a **C# MonoGame** project designed for learning game development concepts, with the primary goal of building a side-scrolling beat 'em up game in the vein of 90s classics like *Streets of Rage*, *Final Fight*, and *Double Dragon*. As part of this effort, we are developing a generic, reusable core engine library (`MonoGameLearning.Core`) that will be useful for bootstrapping future game projects.

The project targets **.NET 10.0** and utilizes **MonoGame.Framework.DesktopGL** for cross-platform desktop support, relying heavily on **MonoGame.Extended** for utilities like cameras, sprites, and input handling.

The solution is structured into two main projects:

1. **`MonoGameLearning.Core`**: A library containing reusable engine-level components, base classes, and utilities.
2. **`MonoGameLearning.Game`**: The main executable project containing specific game logic, assets, and the game loop.

## Architecture

* **Core Library (`MonoGameLearning.Core`)**
  * **`GameCore`**: A base class (inheriting from `Microsoft.Xna.Framework.Game`) that handles boilerplate setup: `GraphicsDeviceManager`, `SpriteBatch`, `ContentManager`, and an `OrthographicCamera` with a `BoxingViewportAdapter` for resolution independence.
  * **`Input`**: Contains `InputManager` to abstract raw input into game actions (e.g., `Action1Pressed`).
  * **`Entities`**: Defines base entity classes.
  * **`Drawing`**: Basic shape drawing utilities.

* **Game Project (`MonoGameLearning.Game`)**
  * **`GameLoop`**: Inherits from `GameCore`. Implements the specific game logic (`Update`, `Draw`), manages entities (like the player), and handles the main application lifecycle.
  * **`Program.cs`**: The entry point, using C# top-level statements to bootstrap `GameLoop`.
  * **`Entities`**: Game-specific entities, such as `PlayerEntity`.
  * **`Sprites`**: Sprite management and animation logic (e.g., `PlayerSprite`).
  * **`Content`**: Contains game assets (images, fonts, etc.) processed by the MonoGame Content Pipeline (`.mgcb`).

## Key Technologies

* **Language**: C# (NET 10.0)
* **Framework**: MonoGame (DesktopGL 3.8.*)
* **Extensions**: MonoGame.Extended (5.3.1), Stateless (5.20.0)

## Building and Running

### Prerequisites

* .NET 10.0 SDK
* MonoGame Content Builder (if modifying assets)

### Commands

**Run the Game:**

```bash
dotnet run --project MonoGameLearning.Game/MonoGameLearning.Game.csproj
```

**Build the Solution:**

```bash
dotnet build
```

**Run Tests:**

```bash
dotnet test
```

## Development Conventions

* **Separation of Concerns**: Keep generic, reusable logic in `Core` and specific game implementation in `Game`.
* **State Management**: The project uses the `Stateless` library for managing entity states.
* **Input**: Input is decoupled from logic via `InputManager` events.
* **Resolution**: The game uses a virtual resolution (`GAME_WIDTH`, `GAME_HEIGHT`) scaled to the window size using `BoxingViewportAdapter`.
* **Conciseness**: Responses and suggestions should include code that is as concise and terse as possible.
* **Modern C#**: Always use the latest C# features (e.g., primary constructors, collection expressions, raw string literals) to ensure the codebase remains modern and idiomatic.
* **Solution Simplification**: Before proposing a solution, ALWAYS include a consideration step to see if the proposed architecture or implementation can be further simplified, refactored, or streamlined. When creating plans, prioritize simplicity — actively seek out and suggest simplifying constraints that reduce code surface area, remove unnecessary abstractions, or collapse parallel structures. Every plan should explicitly consider what can be removed or constrained, not just what needs to be built.
* **Build Verification**: Always run `dotnet build` to ensure the project compiles successfully after any code modifications.
* **Testing**: Always run `dotnet test` to execute unit tests after making any changes to verify no regressions were introduced.
* **Mandatory Pre-Completion Checklist**: Before marking any implementation task as complete, the following steps MUST be performed in order:
  1. Write unit/integration tests covering all new or modified logic.
  2. Run `dotnet build` to verify compilation.
  3. Run `dotnet test` to verify all tests pass with no regressions.
  4. If any step fails, fix the issue before proceeding.
* **Preventing Game-Breaking Bugs (Test Requirement)**: Always write new unit/integration tests when modifying logic — this is **not optional**. Focus tests on critical gameplay failure modes such as:
  * **Out-of-Bounds**: Characters or entities slipping outside of screen, level, or walkable boundaries.
  * **Connectivity & Seams**: Disconnected backgrounds or levels that trap players or break scrolling.
  * **State Machine Deadlocks**: Entities getting stuck in non-interruptible states (e.g., infinite attacking or falling) without recovery.
  * **Collision Failures**: Entities passing through solid boundaries or failing to register collision responses.
* **Camera Tracking**: The camera tracking system losing track of player coordinates or clamping to incorrect screen areas.
* **Diagnostic Debug Warnings**: When adding logic with edge cases, invariants, or states that should never be reached, add `Debug.WriteLine` and/or `Debug.Assert` calls to surface unexpected conditions during local debugging. These are compiled out in Release builds (zero runtime cost) but catch sentinel drift, skipped frames, null invariants, and other game state corruptions during development. Use this pattern:

  ```csharp
  Debug.Assert(condition, "Description of what went wrong");
  if (unexpectedCondition)
      Debug.WriteLine($"[{name}] Descriptive warning — root cause hint");
  ```

* **Debug-Mode Drawing**: When planning or implementing any new system, always consider what should be drawn in debug mode (`IsDebug`). This includes: spatial markers (trigger zones, bounds, spawn points), state indicators (active wave index, enemy count), and any runtime data that aids diagnosis during development. Add debug drawing alongside the feature — not as an afterthought.
* **GC Optimization (Zero-Allocation Gameplay)**: All gameplay-critical paths (`Update`, `Draw`, collision detection, input handling) must be allocation-free to avoid GC-induced frame stutters. Follow these rules:
  * **Pool/reuse allocations** — Use object pools (`ArrayPool<T>`, `Queue<T>`, or custom pools) for transient entities, particles, projectiles, and temporary lists.
  * **Avoid LINQ in hot paths** — LINQ allocates enumerators and closures. Prefer `for`/`foreach` loops with pre-allocated buffers.
  * **Avoid `params` in hot paths** — `params` arrays allocate on every call. Use explicit overloads or `ReadOnlySpan<T>`.
  * **Use `struct` where appropriate** — Prefer `readonly struct` for small, frequently-created data types (e.g., vectors, hitbox results, damage info) to eliminate heap pressure and reduce GC scans.
  * **Pre-allocate buffers** — Use `ArrayPool<byte>` or pre-sized `List<T>` with `Capacity` for serialization, network I/O, and temporary geometry.
  * **Cache delegates and lambdas** — Store static/instance method references in fields; never allocate new lambdas per frame (e.g., don't write `list.ForEach(x => ...)` in Update).
  * **Profile allocations** — Run with `DOTNET_gcServer=1` and monitor GC pause times during development. Flag any unexpected per-frame allocations in code review.
* **Ask Questions When Coding**: Before implementing any design or architecture change, pause to ask the user clarifying questions. Do not silently implement ambiguous or multi-interpretation requests. If a requirement, edge case, or design decision is underspecified, present concrete options and ask for direction. This applies to test strategy, abstraction boundaries, naming, file placement, and any choice that would be costly to reverse.

## MonoGame.Extended Pitfalls

### `AnimatedSprite.Controller` is replaced by `SetAnimation()`

`MonoGame.Extended.AnimatedSprite.Controller` has a public setter. Calling `SetAnimation()` may replace the `Controller` property with a **new** `IAnimationController` instance. This means:

* **Event subscriptions must happen AFTER `SetAnimation()`**, not once at construction time. Subscribing to `Sprite.Controller.OnAnimationEvent` in the constructor subscribes to the *initial* controller, which becomes orphaned after the first `SetAnimation()` call. Events from the new controller (including `AnimationCompleted`) will never fire.
* **Always subscribe/unsubscribe in pairs** around `SetAnimation()` calls for non-looping animations that need completion detection. See `PlayerEntity.SubscribeToAnimationEvent()` / `UnsubscribeFromAnimationEvent()` for the pattern.
* The affected entry/exit callbacks are: `OnAttackingEntry/Exit`, `OnHurtEntry/Exit`, `OnDyingEntry/Exit` — any state that plays a non-looping animation requiring a completion trigger.

---
> Source: [mattemoore/MonoGame_Learning](https://github.com/mattemoore/MonoGame_Learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
