## popp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Deep-Dive Docs

Topic-specific narrative docs live under `docs/`. Read the relevant one BEFORE editing the matching subsystem — they explain the *why* and the non-obvious constraints that aren't visible from the code alone.

- `docs/01-camera-preview.md` — `SurfaceRequest` ↔ `CameraXViewfinder` bridge, `LifecycleStartEffect` rebind keys (Photo↔Video does NOT rebind), `CameraInfo.torchState`/`zoomState` observers.
- `docs/02-photo-and-video-capture.md` — `ImageCapture.OutputFileOptions` + `MediaStoreOutputOptions` paths through `MediaStoreGateway`, `POPP_IMG_` / `POPP_VID_` prefix contract, gallery `LIKE` filter.
- `docs/03-concurrent-camera.md` — `ConcurrentCamera.SingleCameraConfig` × 2 sharing ONE `UseCaseGroup`, `CompositionSettings` fill vs PiP, 720p cap, no `ImageCapture` in dual. **Superseded by doc 06 in current code** — kept for historical context.
- `docs/04-concurrent-video-recording.md` — composited Preview surface IS the encoder input, single MP4 output, why no `MediaMuxer`/FFmpeg needed. **Superseded by doc 06 in current code** — kept for historical context.
- `docs/05-permissions.md` — Accompanist `MultiplePermissionsState` gate, `PermissionEnum` sealed-style queue, optional-perm drop-on-decline, two SDK-split lists must stay in sync.
- `docs/06-live-dual-swap.md` — GLES + MediaCodec composite pipeline replacing `ConcurrentCamera`. Two `SurfaceTexture`s, one shader, two eglSurfaces (preview + encoder) sharing one EGL context. Live primary/PiP swap mid-recording lands seamlessly in saved MP4. `DualLayout` enum cycles composite variants (`PIP_RECT`, `PIP_CIRCLE`, `SPLIT_VERTICAL`, `SPLIT_HORIZONTAL`, `REACTION_STRIP`) via tap; blocked while recording. Drag/pinch PiP placement via `PipDragOverlay` writes `@Volatile` renderer fields off the recomposition path. Files: `ui/features/camera/dual/{DualCameraRenderer, DualCameraSession, DualVideoEncoder, DualPreviewSurface, PipDragOverlay, DualLayout}.kt`.
- `docs/07-usecase-architecture.md` — Per-action use cases + `CameraSessionRepository` + `CameraProviderRepository` + `MediaStoreGateway`. ViewModels are UI orchestrators only; all blocking work hops to `@IoDispatcher` inside use cases; CameraX binds hop to `Dispatchers.Main.immediate`. `@ApplicationContext` is injected in only two places (`DefaultCameraProviderRepository`, `MediaStoreGateway`); use cases never take raw `Context`. DI: `di/DispatcherModule.kt`, `di/CameraDataModule.kt`, qualifiers in `di/qualifiers/Dispatchers.kt`.

When introducing a new subsystem worth a deep-dive (e.g. gallery internals, Navigation3 backstack, theming), add a new doc under `docs/` and link it here.

## Build & Test

Single-module Gradle project. Use the wrapper.

```bash
./gradlew assembleDebug              # Build debug APK
./gradlew installDebug               # Install on attached device/emulator
./gradlew lint                       # Android lint
./gradlew test                       # JVM unit tests (app/src/test)
./gradlew connectedDebugAndroidTest  # Instrumented tests (needs device)

# Single test class / method (JVM)
./gradlew :app:testDebugUnitTest --tests "com.debanshu777.popp.SomeTest"
./gradlew :app:testDebugUnitTest --tests "com.debanshu777.popp.SomeTest.methodName"
```

Toolchain pins (see `gradle/libs.versions.toml`): Kotlin 2.3.0, AGP 9.0.0, JDK 21, compileSdk 36, minSdk 24. Heavy reliance on alpha/preview artifacts (Compose BOM 2026.01.00, Material3 1.5.0-alpha12, Navigation3 1.0.0, CameraX 1.6.0-alpha02). KSP for Hilt codegen.

## Architecture

Single-Activity Compose app. Entry = `MainActivity` → `PoppTheme` → permission gate → `MainContent` (Navigation3 backstack).

**Permission gate.** `MainActivity.onCreate` builds the required-permission list (CAMERA + RECORD_AUDIO + media perms split by SDK at TIRAMISU) and renders `PermissionsScreen` until `allPermissionsGranted`. Only then is `MainContent` composed. Adding a new runtime permission = update the `buildList` block in `MainActivity.kt` AND `AndroidManifest.xml`.

**Navigation3.** `MainContent` uses `rememberNavBackStack` + `NavDisplay`. Destinations are `@Serializable data object`s under `navigation/NavigableScreen.kt`. Backstack survives state restoration via a `SavedStateConfiguration` with a `SerializersModule` polymorphic block declared at `MainActivity.kt:40-48`. **Any new `NavigableScreen` subclass MUST be registered in that polymorphic block** or restoration will crash. Helpers in `navigation/NavigationExtensions.kt` (`replaceLast`, `popIfPossible`).

**DI.** `PoppApplication` is `@HiltAndroidApp`. Hilt modules:
- `di/MediaModule.kt` — binds `LocalMediaRepository → MediaRepository` (`@Singleton`).
- `di/CameraDataModule.kt` — binds `DefaultCameraProviderRepository → CameraProviderRepository` and `DefaultCameraSessionRepository → CameraSessionRepository` (`@Singleton`).
- `di/DispatcherModule.kt` — provides `@IoDispatcher`/`@DefaultDispatcher CoroutineDispatcher` (qualifiers in `di/qualifiers/Dispatchers.kt`).

Use cases are auto-injected via `@Inject constructor` — no module needed. `MediaStoreGateway` (`data/media/`) is also `@Inject constructor` + `@Singleton`, no module binding. ViewModels are `@HiltViewModel` and consumed in Composables via `hiltViewModel()`. `@ApplicationContext Context` injection is limited to `DefaultCameraProviderRepository`, `MediaStoreGateway`, and `LocalMediaRepository` — use cases never inject raw `Context`.

**Camera pipeline.** UI orchestration in `ui/features/camera/CameraViewModel.kt`; all hardware/session work in `ui/features/camera/usecase/*.kt` use cases backed by `data/camera/CameraSessionRepository` (singleton flow of live handles) and `data/camera/CameraProviderRepository` (thin wrapper around CameraX `ProcessCameraProvider`). Use cases own threading: blocking work `withContext(@IoDispatcher)`, CameraX binds `withContext(Dispatchers.Main.immediate)`. ViewModel stays on `viewModelScope`. `CameraScreen` drives lifecycle through `LifecycleStartEffect(useFront, useDualCamera)`. Snackbar messages flow through a `Channel(CONFLATED)`. Modes (`Photo`/`Video`) defined in `CameraMode.kt`; the FAB action switches on `selectedMode`. See `docs/06-live-dual-swap.md` for the GL composite pipeline and `docs/07-usecase-architecture.md` for the use-case layout.

**Media persistence convention.** Captured files are written to `DCIM/Popp/` via `MediaStore` with names prefixed by `MediaConstants.IMAGE_PREFIX` (`POPP_IMG_`) / `VIDEO_PREFIX` (`POPP_VID_`). `LocalMediaRepository` queries MediaStore with a `DISPLAY_NAME LIKE 'PREFIX%'` selection so the in-app gallery shows ONLY app-captured media — anything written without those prefixes will be invisible to the gallery. Preserve the prefixes exactly when changing capture code.

**Gallery.** `GalleryViewModel` invokes `LoadMediaUseCase` (wraps `MediaRepository.getAllMedia()` on `@IoDispatcher`), then groups via `DateUtils` extensions (`groupByDate` / `groupByMonth`) keyed by header strings (`"Today"`, `"Yesterday"`, `"MMMM d, yyyy"`, `"MMMM yyyy"`). `DeleteMediaUseCase` handles row removal. View mode toggle (`GalleryViewMode.DAILY|MONTHLY`) re-groups in-memory without re-querying. `GalleryScreen` renders a `LazyColumn` of `stickyHeader` + `MediaGridSection` per group.

**Theming.** `ui/theme/Theme.kt` uses MaterialKolor `rememberDynamicColorScheme(seed = 0xFF4285F4, isAmoled = true)` wrapped in `MaterialExpressiveTheme` with `MotionScheme.expressive()`. A `SharedTransitionLayout` is installed at the theme root and exposed via `LocalSharedTransitionScope` for shared-element transitions across screens.

## Package Layout

```
com.debanshu777.popp
├── MainActivity, PoppApplication
├── data
│   ├── camera        — CameraProviderRepository, CameraSessionRepository, CameraSessionState, DualPreviewTarget
│   └── media         — MediaRepository, LocalMediaRepository, MediaStoreGateway, MediaDescriptor, MediaConstants
├── di                — Hilt modules (MediaModule, CameraDataModule, DispatcherModule, qualifiers/)
├── navigation        — NavigableScreen, NavigationExtensions
└── ui
    ├── features
    │   ├── camera    — CameraScreen, CameraViewModel, CameraUiState, CameraMode, dual/, usecase/
    │   ├── gallery   — GalleryScreen, GalleryViewModel, GalleryUiState, usecase/
    │   └── permissions
    └── theme         — PoppTheme, Color, Type
```

---
> Source: [Debanshu777/Popp](https://github.com/Debanshu777/Popp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
