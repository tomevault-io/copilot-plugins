## component-framework

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
./gradlew assembleDebug          # Build debug APK (app) + library AAR
./gradlew assembleRelease        # Build release AAR
./gradlew test                   # Run unit tests
./gradlew lint                   # Static analysis
./gradlew clean build            # Clean + full build
```

## Project Structure

Root project is the library itself (single-module for JitPack), with one submodule:

- **Root (`:`)** — Core framework library. All source in `src/main/java/com/andy/modularization/`. Published to JitPack as `com.github.andyhaha:component-framework`.
- **`:app`** — Demo application showing Activity-scoped and Fragment-scoped component usage.

## Architecture

This is a lifecycle-aware component framework for Android. The key design:

**Entry point:** `componentScope { }` DSL (defined in `ComponentKt.kt`) — called from `Activity.onCreate()` or `Fragment.onViewCreated()`. It wires up the host adapter, scope, container, and lifecycle observer in one call.

**Lifecycle flow:**
1. `componentScope` creates a `ComponentHost` (adapter over Activity/Fragment) and a `ComponentScope` (DSL receiver + service registry)
2. Inside the DSL block, `+Component(host, binding)` (unary plus operator on `ComponentScope`) instantiates components and auto-registers any `Service` interfaces they implement
3. After the block, `ComponentContainer` topologically sorts components based on `dependencies()` declarations (DFS with cycle detection)
4. `ComponentContainer` observes the host lifecycle and propagates `onCreate`→`onStart`→`onResume`→`onPause`→`onStop`→`onDestroy` to all components in order

**Inter-component communication:** Components implement `Service` sub-interfaces and call `getService<T>()` to find other components' services. Services are scoped per `ComponentScope` instance (no global state).

**State machine:** `Component` enforces a strict state machine (`ComponentState` enum: INITIALIZED→CREATED→STARTED→RESUMED↔PAUSED→STOPPED→DESTROYED). Each `perform*` method guards against invalid transitions.

**Error isolation:** Each lifecycle callback dispatch in `ComponentContainer` is wrapped in try-catch so one component's failure doesn't crash others.

## Key Framework Classes (all in `src/main/java/com/andy/modularization/`)

| Class | Purpose |
|---|---|
| `Component.kt` | Base class — lifecycle callbacks, state machine, `dependencies()` declaration |
| `ComponentHost.kt` | Interface abstracting Activity/Fragment (provides lifecycle, context, ViewModelStore) |
| `ComponentKt.kt` | `componentScope()` DSL extensions + `getService<T>()` extension |
| `ComponentScope.kt` | DSL receiver (`+` operator), scope-local service registry |
| `ComponentContainer.kt` | Holds components, topological sort, lifecycle propagation |
| `ActivityComponentHost.kt` / `FragmentComponentHost.kt` | Host adapters using Kotlin delegation |

## Conventions

- **Kotlin** throughout, targeting Java 11 bytecode
- **View binding** enabled in app module — components receive binding objects in constructors
- **Service interfaces** extend the `Service` marker interface and use the naming convention `*Service` (e.g., `TitleBarService`)
- **Dependencies** declared by overriding `dependencies(): List<KClass<out Service>>` returning service interface KClasses
- **Version catalog** at `gradle/libs.versions.toml` manages all dependency versions
- Android minSdk 23, compileSdk/targetSdk 36

---
> Source: [andyhaha/component-framework](https://github.com/andyhaha/component-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
