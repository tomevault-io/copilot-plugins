## glowrancher

> - ALWAYS read `openspec/specs/` at the start of every session

# Unity Project — Gemini CLI Context

## 🧠 Session Start Protocol
- ALWAYS read `openspec/specs/` at the start of every session
- ALWAYS read `openspec/changes/` to understand work in progress
- Summarize what specs you found before proceeding

## 🎮 Project
- Engine: Unity 2022.2 LTS (URP)
- Language: C# (.NET Standard 2.1)
- Architecture: Component-based (MonoBehaviour)

## 📐 OpenSpec Rules
- NEVER write code before a spec exists in `openspec/specs/`
- If asked to implement something without a spec, suggest running `/opsx:propose` first
- After implementation, remind me to run `/opsx:archive`
- When refactoring, only `design.md` should change — never the behaviour spec

## ⚠️ Hard Rules (Non-Negotiables)
- NEVER modify `.unity` scene files directly
- NEVER edit files inside `Packages/`
- NEVER modify `ProjectSettings/` unless explicitly asked
- NEVER add code to existing scripts without reading them first and asking permission

## 🏛️ SOLID Principles
Apply these on every script you write or refactor:

- **S — Single Responsibility**: Each class does ONE thing only.
  - ✅ `PlayerInput.cs`, `PlayerMovement.cs`, `PlayerAnimator.cs` as separate classes
  - ❌ One `Player.cs` that handles input, physics, animation and health

- **O — Open/Closed**: Open for extension, closed for modification.
  - Prefer adding new classes over editing existing ones
  - Use abstract base classes or interfaces for extensible behaviour

- **L — Liskov Substitution**: Subtypes must be substitutable for their base types.
  - If `EnemyBase` has a `TakeDamage()` method, all enemies must honour it correctly
  - Never override a method to do nothing or throw exceptions

- **I — Interface Segregation**: No class should depend on interfaces it doesn't use.
  - Split large interfaces: `IDamageable`, `IMovable`, `IInteractable` separately
  - ❌ One fat `IEntity` interface with 10 methods most classes ignore

- **D — Dependency Inversion**: Depend on abstractions, not concretions.
  - Inject dependencies via constructor or `[SerializeField]`
  - ❌ `GetComponent<ConcreteClass>()` inside logic — use interfaces instead

## 🎨 Design Patterns
Use these patterns when the problem fits — never force them:

### Creational
- **Factory**: Use to instantiate enemies, projectiles, VFX, UI elements
  - Prefer `ScriptableObject`-based factories in Unity
  - ❌ Never use `new` or raw `Instantiate()` scattered across gameplay code

- **Object Pool**: ALWAYS use for frequently spawned/destroyed objects
  - Bullets, particles, enemies, damage numbers
  - Use Unity's built-in `ObjectPool<T>` (UnityEngine.Pool) — don't reinvent it

- **Singleton**: Use sparingly, only for true global systems
  - Acceptable: `AudioManager`, `GameManager`, `SceneLoader`
  - ❌ Never Singleton a gameplay class like `Player` or `Enemy`

### Behavioural
- **Observer (Event-driven)**: Use for decoupled communication between systems
  - Prefer C# `Action`/`event` or Unity `UnityEvent`
  - Use `ScriptableObject`-based events for cross-scene communication
  - ✅ `OnPlayerDied`, `OnScoreChanged`, `OnEnemySpawned`

- **State**: Use for objects with clearly distinct modes of behaviour
  - Player states: `IdleState`, `RunState`, `JumpState`, `DashState`
  - Implement via `IState` interface with `Enter()`, `Tick()`, `Exit()`
  - ❌ Avoid giant `switch` or `if/else` chains in `Update()`

- **Command**: Use for input handling, undo systems, replay
  - Wrap each action as an object: `MoveCommand`, `JumpCommand`
  - Enables input rebinding and action queuing cleanly

- **Strategy**: Use to swap algorithms or behaviours at runtime
  - e.g. different AI movement strategies: `PatrolStrategy`, `ChaseStrategy`
  - Inject via interface: `IMovementStrategy`

### Structural
- **MVP (Model-View-Presenter)**: Use for UI systems
  - Model = data (ScriptableObject or plain C# class)
  - View = MonoBehaviour that only updates visuals
  - Presenter = logic that connects Model → View
  - ❌ Never put game logic inside UI MonoBehaviours

- **Flyweight**: Use for shared data across many instances
  - Enemy stats, item definitions → use `ScriptableObject` as shared data
  - Instances reference the SO, never duplicate the data

## 🧱 C# Conventions
- Prioritize using `[SerializeField] private` instead of `public` for Inspector fields
- Always use `Rigidbody2D` for physics — never move via `Transform` directly
- Namespace: `MyGame.{Module}` (e.g. `MyGame.Player`)
- One MonoBehaviour per file, filename matches class name

## 🐛 Bug Handling
- Audit the relevant script before suggesting a fix
- State clearly: where the bug is, why it happens, how to fix it
- Do NOT touch unrelated code

## ✅ Spec Compliance Check
- If I say "check compliance", compare my code against `openspec/specs/` and flag any deviations
- If I say "check patterns", review the code for SOLID violations or pattern misuse

# 📁 Project Structure
- Scripts: `Assets/Code/Scripts/`
- ScriptableObjects: `Assets/Code/ScriptableObjects/`
- Prefabs: `Assets/Level/Prefabs/`

### Conventions
- ALWAYS place new scripts in `Assets/Code/Scripts/{Module}/`
  (e.g. `Assets/Code/Scripts/Player/`, `Assets/Code/Scripts/Enemy/`)
- ALWAYS place new ScriptableObjects in `Assets/Code/ScriptableObjects/{Module}/`
- ALWAYS place new Prefabs in `Assets/Level/Prefabs/{Module}/`
- NEVER create scripts in the root `Assets/` folder
- NEVER place scripts outside of `Assets/Code/Scripts/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/victorcn2006)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/victorcn2006)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
