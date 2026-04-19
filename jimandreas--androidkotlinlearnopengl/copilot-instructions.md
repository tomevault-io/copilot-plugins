## androidkotlinlearnopengl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Android Kotlin app implementing [learnopengl.com](https://learnopengl.com) tutorials adapted for Android/OpenGL ES 3+. Written in Kotlin with Jetpack architecture components.

## Build Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Run unit tests (JUnit 5 via useJUnitPlatform)
./gradlew test

# Run a specific unit test class
./gradlew test --tests "org.rajawali3d.math.Matrix4Test"

# Run instrumented tests (requires a connected device or emulator)
./gradlew connectedAndroidTest

# Run a specific instrumented test class
./gradlew connectedAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=org.rajawali3d.math.QuaternionTest

# Run lint
./gradlew lint
```

Unit tests (`src/test`): `Matrix4Test`, `Vector3Test` — use JUnit 5 Jupiter annotations.
Instrumented tests (`src/androidTest`): `QuaternionTest` — runs on-device via `AndroidJUnitRunner`.

**Important:** Build the project before reading code closely in Android Studio. Data Binding and Navigation SafeArgs generate code at build time — generated classes like `*Binding`, `*Directions`, `*Args`, and `*FragmentArgs` will show as unresolved until after a build.

## Architecture

### Navigation Flow
`SelectorFragment` (exercise list) → user taps button → `NavController` navigates with exercise name string → `ExerciseFragment` instantiates the matching Renderer.

### Key Layers

**UI** (`com.androidkotlin.opengl.ui`):
- `MainActivity` — single activity; sets content view with NavHost
- `SelectorFragment` — lists exercises; buttons post to `ViewModel.buttonClicked` LiveData; observes it and navigates via `SelectorFragmentDirections.actionShowExercise(it)`
- `ExerciseFragment` — receives `selectedExercise` string arg (via SafeArgs); switches on it to instantiate the correct Renderer; manages GLSurfaceView lifecycle (`onPause`/`onResume`)
- `ViewModel` — shared LiveData: `wireFrameModeOn`, `shaderToggle`, `buttonClicked`, `lowResViewAdjuster`; buttons are bound via DataBinding

**Rendering** (`com.androidkotlin.opengl.realtime`):
- `RendererBaseClass` — base class holding `@Volatile` touch state fields (`deltaX`, `deltaY`, `deltaTranslateX`, `deltaTranslateY`, `deltaZoom`, `scaleCurrentF`) shared between UI and GL threads
- `SurfaceViewInstance` (extends `GLSurfaceView`) — handles 1-finger rotate and 2-finger pan/zoom touch events; writes to `RendererBaseClass` volatile fields; switches between `RENDERMODE_CONTINUOUSLY` and `RENDERMODE_WHEN_DIRTY`

**Exercise Renderers** (`com.androidkotlin.opengl.learnopengl` and `googlesample`):
Each renderer extends `RendererBaseClass` and implements `GLSurfaceView.Renderer` (`onSurfaceCreated`, `onSurfaceChanged`, `onDrawFrame`). Named after the learnopengl.com exercise number, e.g. `Renderer174Camera`, `Renderer4103AdvancedAsteroidsInstanced`.

**Utilities** (`com.androidkotlin.opengl.util`):
- `Shader` — loads GLSL from assets or strings, compiles, links; exposes `use()`, `setInt()`, `setFloat()`, `setVec3()`, `setMat4()`, etc.
- `Camera` — Kotlin translation of learnopengl's `camera.h`; uses Rajawali Quaternion for view matrix; `setRotation()`, `moveRight()`, `moveForward()`, `zoomInOut()`
- `ObjFile` — parses `.obj`/`.mtl` files from assets
- `LoadTextureFromAsset` — loads PNG textures from assets into OpenGL

**Math Library** (`org.rajawali3d.math`):
Rajawali math modules ported to Kotlin. Key classes: `Matrix4`, `Quaternion`, `Vector3`, `Vector2`, `Matrix`, `Plane`, `WorldParameters`. Tests live in `src/test` and `src/androidTest`.

### Shader Files
GLSL shaders are stored as plain text in `app/src/main/assets/` (e.g. `7.4.camera.vs`, `7.4.camera.fs`). The `Shader` class loads them via `context.assets.open()`.

### Adding a New Exercise
1. Add a new Renderer class in `learnopengl/` that extends `RendererBaseClass` and implements `GLSurfaceView.Renderer`
2. Add a button in `SelectorFragment`'s layout that calls `viewModel.onButton("YourExerciseName")`
3. Add a `when` branch in `ExerciseFragment.onCreateView()` matching that string

## Emulator Configuration

For OpenGL ES 3+ in the emulator, add to `~/.android/advancedFeatures.ini`:
```
GLESDynamicVersion = on
```

## Dependencies and Versions

Managed via `gradle/libs.versions.toml`. Key versions: AGP 9.0.1, compileSdk 36, minSdk 24, Java/Kotlin JVM 17, Navigation 2.9.7, Lifecycle 2.10.0, JUnit Jupiter 6.0.3.

No third-party graphics libraries — only the Android Framework OpenGL ES APIs. Rajawali math is vendored directly into `org.rajawali3d.math`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimandreas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
