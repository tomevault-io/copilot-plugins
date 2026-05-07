## stratix

> This file is the primary working contract for any agent operating in this repository. Read it before touching any file. Subdirectory `AGENTS.override.md` files extend and narrow these rules for their specific scope.

# AGENTS.md — Stratix

This file is the primary working contract for any agent operating in this repository. Read it before touching any file. Subdirectory `AGENTS.override.md` files extend and narrow these rules for their specific scope.

**Modernization contract reference:** For any work governed by the structural modernization program, read `Docs/Stratix_Modernization_Contracts.md` together with `Docs/Stratix_Modernization_Plan.md`, `Docs/Stratix_Monolith_Breakdown.md`, and `Docs/Stratix_File_Matrix.md`. The contracts document is the canonical AGENTS reference for the Floor Contract and Execution Contract.

**Skill routing reference:** For any coding, review, build, test, performance, rendering, persistence, or platform API task in this repo, use `[$ios-skills:ios-skills-router](/Users/nicholas/.ios-skills/skills/_router/SKILL.md)` first and then load the repo-relevant skills listed in `Docs/Stratix_Skill_Policy.md` before coding.

---

## Project Overview

**Stratix** is a tvOS 26 SwiftUI application that provides an Xbox cloud gaming client (xCloud / Game Pass) for Apple TV. It is the single app target in this monorepo. The repo also contains seven Swift Package Manager packages that the app target consumes.

- **App target:** `Apps/Stratix/` (product name: Stratix, brand: STRATIX)
- **Packages:** `Packages/` — DiagnosticsKit, StratixCore, StratixModels, InputBridge, StreamingCore, VideoRenderingKit, XCloudAPI
- **Third-party:** `ThirdParty/WebRTC/WebRTC.xcframework` — pre-built binary, tvOS arm64 + simulator
- **Tools:** `Tools/webrtc-build/` — build scripts and tvOS-specific patches for the WebRTC source

The app streams cloud games over WebRTC. The UI layer is SwiftUI throughout. The rendering layer uses Metal. Audio is managed via AVAudioSession with tvOS-specific patches applied to the WebRTC framework.

---

## Build Posture

- **Swift version:** 6.2
- **Strict concurrency checking:** complete (enforced for all targets)
- **Deployment target:** tvOS 26.0
- **Xcode workspace:** `Stratix.xcworkspace`

These are hard constraints. Do not lower the deployment target. Do not weaken the concurrency checking level. Do not add `nonisolated(unsafe)` or `@unchecked Sendable` without a documented reason in the file.

**Available build schemes (in Stratix.xcworkspace/xcshareddata/xcschemes/):**

| Scheme | Purpose |
|--------|---------|
| `Stratix-Debug` | Default development build |
| `Stratix-Profile` | Instruments profiling |
| `Stratix-MetalProfile` | GPU profiling via Metal |
| `Stratix-Perf` | Performance test runs |
| `Stratix-ShellUI` | UI test harness mode |
| `Stratix-ReleaseRun` | Release-configuration local run |
| `Stratix-Packages` | Runs all SPM package tests |
| `Stratix-Validation` | Full validation suite |

**WebRTC compile flag:** `OTHER_SWIFT_FLAGS = -DWEBRTC_AVAILABLE`. All WebRTC-dependent code is wrapped in `#if WEBRTC_AVAILABLE`. Do not remove these guards.

---

## Skill Policy

Agents working in this repo should use repo-matched skills deliberately, not opportunistically after coding has already started.

### Default skill stack

Load these by default for most source changes:

- `[$ios-skills:ios-skills-router](/Users/nicholas/.ios-skills/skills/_router/SKILL.md)`
- `[$ios-skills:swiftui-pro](/Users/nicholas/.ios-skills/skills/twostraws--swiftui-pro/SKILL.md)` for app-target UI and SwiftUI composition
- `[$ios-skills:swift-concurrency-pro](/Users/nicholas/.ios-skills/skills/twostraws--swift-concurrency-pro/SKILL.md)` for all async, actor, and boundary-isolation work
- `[$ios-skills:swift-coding-guideline](/Users/nicholas/.ios-skills/skills/martinlasek--swift-coding-guideline/SKILL.md)` for general Swift/package changes
- `[$ios-skills:swift-accessibility-skill](/Users/nicholas/.ios-skills/skills/pasqualevittoriosi--swift-accessibility/SKILL.md)` whenever touching app UI

### Repo-specific trigger skills

Use these when the file or domain matches:

- `[$ios-skills:swift-testing-expert](/Users/nicholas/.ios-skills/skills/avdlee--swift-testing-expert/SKILL.md)` for new package tests and Swift Testing migrations
- `[$ios-skills:swiftdata](/Users/nicholas/.ios-skills/skills/dpearson2699--swiftdata/SKILL.md)` for `Packages/StratixCore/Sources/StratixCore/Hydration/SwiftDataLibraryRepository.swift` and nearby hydration persistence work
- `[$ios-skills:ios-networking](/Users/nicholas/.ios-skills/skills/dpearson2699--ios-networking/SKILL.md)` and `[$ios-skills:swift-codable](/Users/nicholas/.ios-skills/skills/dpearson2699--swift-codable/SKILL.md)` for `Packages/XCloudAPI/`
- `[$ios-skills:swift-security-expert](/Users/nicholas/.ios-skills/skills/ivan-magda--swift-security-expert/SKILL.md)` for auth and token storage, especially `Packages/XCloudAPI/Sources/XCloudAPI/Auth/TokenStore.swift`
- `[$ios-skills:debugging-instruments](/Users/nicholas/.ios-skills/skills/dpearson2699--debugging-instruments/SKILL.md)` for streaming, rendering, AVFoundation, performance, and runtime investigations
- `[$ios-skills:swiftui-view-refactor](/Users/nicholas/.ios-skills/skills/dimillian--swiftui-view-refactor/SKILL.md)` or `[$ios-skills:swiftui-performance-audit](/Users/nicholas/.ios-skills/skills/dimillian--swiftui-performance-audit/SKILL.md)` for large CloudLibrary shell/view work
- `[$ios-skills:swiftui-uikit-interop](/Users/nicholas/.ios-skills/skills/dpearson2699--swiftui-uikit-interop/SKILL.md)` for UIKit bridge surfaces in streaming/rendering
- `[$ios-skills:swiftui-liquid-glass](/Users/nicholas/.ios-skills/skills/dimillian--swiftui-liquid-glass/SKILL.md)` when changing tvOS 26 shell chrome/material surfaces
- `[$ios-skills:xcodebuildmcp](/Users/nicholas/.ios-skills/skills/getsentry--xcodebuildmcp/SKILL.md)` or `[$ios-skills:ios-debugger-agent](/Users/nicholas/.ios-skills/skills/dimillian--ios-debugger-agent/SKILL.md)` for simulator/device validation
- `[$ios-skills:cleanse](/Users/nicholas/.ios-skills/skills/andrewgleave--cleanse/SKILL.md)` and `[$ios-skills:orchestrate-batch-refactor](/Users/nicholas/.ios-skills/skills/dimillian--orchestrate-batch-refactor/SKILL.md)` for structural cleanup and large refactor waves

### Build and package analysis skills

Prefer these when the task is about build speed, package overhead, or validation posture:

- `[$ios-skills:xcode-build-benchmark](/Users/nicholas/.ios-skills/skills/avdlee--xcode-build-benchmark/SKILL.md)`
- `[$ios-skills:xcode-compilation-analyzer](/Users/nicholas/.ios-skills/skills/avdlee--xcode-compilation-analyzer/SKILL.md)`
- `[$ios-skills:spm-build-analysis](/Users/nicholas/.ios-skills/skills/avdlee--spm-build-analysis/SKILL.md)`

Use `Docs/Stratix_Skill_Policy.md` for the repo-specific rationale and file/domain mapping.

---

## Setup

1. Open `Stratix.xcworkspace` in Xcode 26+.
2. Select the `Stratix-Debug` scheme and an Apple TV simulator or device.
3. Build. No additional configuration is required for simulator builds.
4. For device builds, the `ThirdParty/WebRTC/WebRTC.xcframework/tvos-arm64/` slice is required and is already present.
5. To rebuild WebRTC from source: see `Tools/webrtc-build/README.md`.

---

## Repository Layout

```
stratix/
├── Apps/
│   └── Stratix/           # App target + tests
│       ├── Sources/Stratix/
│       │   ├── App/                 # Entry point, launch modes
│       │   ├── Auth/                # AuthView, DeviceCodeView
│       │   ├── Consoles/            # Console list + stream launch
│       │   ├── Data/CloudLibrary/   # Side-effect-free data shaping
│       │   ├── Features/
│       │   │   ├── CloudLibrary/    # Largest feature slice
│       │   │   ├── Guide/           # Guide overlay
│       │   │   └── Streaming/       # Stream view, overlay, rendering
│       │   ├── Integration/
│       │   │   ├── Previews/        # Preview data and harness
│       │   │   ├── UITestHarness/   # UI test scaffolding
│       │   │   └── WebRTC/          # Metal renderer + WebRTC client impl
│       │   ├── Models/              # App-local model types
│       │   ├── Profile/             # Profile overlay views
│       │   ├── RouteState/          # Navigation types
│       │   ├── Sections/            # Guide pane sections
│       │   ├── Shared/
│       │   │   ├── Components/      # Reusable UI components
│       │   │   └── Theme/           # StratixTheme, typography, image pipeline
│       │   ├── Shell/               # MainTabView, StratixTabContentView
│       │   └── ViewState/           # CloudLibraryViewState, nav perf tracker
│       ├── StratixTests/  # Unit tests (XCTest / Swift Testing)
│       ├── StratixUITests/
│       ├── StratixPerformanceTests/
│       └── StratixPerformanceUITests/
├── Packages/
│   ├── DiagnosticsKit/
│   ├── StratixCore/
│   ├── StratixModels/
│   ├── InputBridge/
│   ├── StreamingCore/
│   ├── VideoRenderingKit/
│   └── XCloudAPI/
├── ThirdParty/
│   └── WebRTC/WebRTC.xcframework
├── Tools/
│   ├── webrtc-build/
│   └── ci/
└── .github/
    └── workflows/
```

---

## Architecture

### Boot sequence

`StratixApp` (SwiftUI `@main`) creates one `AppCoordinator` from `StratixCore` and injects all of its controllers as typed environment objects:

```
StratixApp
  └── AppCoordinator (StratixCore, @MainActor, ObservableObject)
        ├── sessionController
        ├── libraryController
        ├── profileController
        ├── consoleController
        ├── streamController
        ├── shellBootstrapController
        ├── achievementsController
        ├── inputController
        ├── previewExportController
        └── settingsStore
```

`RootView` reads auth state from `AppCoordinator` and routes to: `AuthView` → `DeviceCodeView` → authenticated shell.

### Shell layers

`AuthenticatedShellView` (in `Shell/`) is the container for the authenticated experience. It wraps `CloudLibraryView` as the primary content. `GamePassShellView`/`MainTabView` provide ambient background and top-chrome overlay.

### CloudLibrary feature (largest feature slice)

```
CloudLibraryView              ← coordinator-connected entry
  └── CloudLibraryShellHost   ← layout + action routing owner
        ├── CloudLibraryShellView       ← side rail + content host
        ├── CloudLibraryContentRouteHost← routes to browse/utility/detail
        ├── CloudLibraryBrowseRouteHost ← home/library/search/consoles
        └── CloudLibraryUtilityRouteHost← profile/settings overlays
```

**State objects (all @MainActor, @Observable):**
- `CloudLibraryViewModel` — source of truth for all library data projections
- `CloudLibrarySceneModel` — scene-level derived state (routes, hero bg, status)
- `CloudLibraryPresentationStore` — presentation projection cache
- `CloudLibraryRouteState` — current browse/utility/detail routes
- `CloudLibraryFocusState` — focus request tokens and settled tile state
- `CloudLibraryLoadState` — loading/error/success envelope

**Data shaping (`Data/CloudLibrary/`):** All transformation from raw library models to view state is side-effect-free and has no async work. These functions are pure and testable without mocks.

### Action routing

`CloudLibraryShellHost` owns the three routed action bags directly through its `browseActions`, `detailActions`, and `utilityActions` computed properties. The former `CloudLibraryActionFactory` passthrough seam is already dissolved.

`CloudLibraryBrowseRouteActions`, `CloudLibraryDetailRouteActions`, and `CloudLibraryUtilityRouteActions` remain the correct boundary types. Keep closures in these routed action structs explicitly `@MainActor` when they are invoked from `@MainActor` UI code.

### Streaming feature

```
StreamView
  └── StreamModalControllerHost
        ├── StreamOverlay (overlay controls + diagnostics)
        └── RenderSurfaceCoordinator
              └── RendererAttachmentCoordinator (@MainActor)
                    ├── MetalVideoRenderer (Integration/WebRTC/)
                    └── SampleBufferDisplayRenderer (series of extensions)
```

`RendererAttachmentCoordinator` is `@MainActor` and manages renderer selection, lifecycle, and fallback. The UIKit/probe support types now live under `Features/Streaming/Rendering/UIKitAdapters/`, leaving the coordinator focused on policy and attachment work.

### Package roles

| Package | Role | Key constraint |
|---------|------|----------------|
| `StratixCore` | App lifecycle, auth, controllers, hydration | Owns `AppCoordinator`. No UI. |
| `StratixModels` | Shared model types | No dependencies on other packages |
| `XCloudAPI` | Network layer — xCloud API, auth, catalog | No UI |
| `StreamingCore` | WebRTC session, SDP/ICE, channels | No Metal, no UIKit |
| `VideoRenderingKit` | Upscale strategy resolution | No UIKit, no Metal directly |
| `InputBridge` | Gamepad input, input queue | No UI |
| `DiagnosticsKit` | Metrics pipeline, logging, telemetry | No UI |

Packages do not depend on the app target. The app target depends on packages. Packages may depend on each other only along the data-flow direction (StratixCore → XCloudAPI, StreamingCore; not the reverse).

### WebRTC integration

`Integration/WebRTC/` is the boundary layer between the WebRTC binary framework and the rest of the app. Everything in this directory is conditionally compiled with `#if WEBRTC_AVAILABLE`. Outside this directory, the app uses protocol abstractions from `StreamingCore` and `StratixCore`.

---

## Concurrency Model (Layered Actor Strategy)

This project uses a **layered actor strategy**. "Everything is MainActor" is not the model. "Nonisolated by default" is not the model.

**Layer 1 — UI and navigation (@MainActor)**
All SwiftUI views, all view models (`CloudLibraryViewModel`, `CloudLibrarySceneModel`, etc.), all focus/route state. These are `@MainActor` and must remain so. Do not add background threading to UI state without explicit justification.

**Layer 2 — Service and cache (isolated actors)**
`RemoteImagePipeline` in `Shared/Theme/RemoteImagePipeline.swift` is a correct example: an `actor` that owns a mutable cache and an in-flight deduplication dictionary. New shared, mutable, cross-cutting state belongs in a named actor, not in a `nonisolated` class protected by `NSLock` unless benchmark evidence requires it.

**Layer 3 — Rendering and compute (explicit concurrency)**
Metal rendering runs off the main thread via `MTKViewDelegate` callbacks. Heavy async work (image decoding, frame processing) must use `@concurrent` when called from `@MainActor` context. Do not mark rendering code `@MainActor` unless it is purely UI.

**Cross-boundary closures:** All action closures that originate in `@MainActor` view code must be annotated `@MainActor (...) -> Void` to make isolation explicit. Do not use untyped `@escaping` closures at `@MainActor` boundaries.

---

## Testing Strategy

### Test targets

| Target | Framework | Scope |
|--------|-----------|-------|
| `StratixTests` | XCTest (migrating to Swift Testing) | Unit tests for state, routing, data shaping, focus, streaming |
| `StratixUITests` | XCTest (XCUI) | Shell checkpoint captures |
| `StratixPerformanceTests` | XCTest | Launch, route switch, post-stream freshness |
| `StratixPerformanceUITests` | XCTest | Performance UI scenarios |
| Per-package `Tests/` | Swift Testing (new) / XCTest (existing) | Package-level unit tests |

### Rules

1. **Swift Testing** (`import Testing`) is the target framework for new pure-Swift unit tests. Migrate existing XCTest unit tests incrementally when touching a file.
2. **XCTest + XCUI** must remain for all UI automation. Swift Testing does not replace XCUI.
3. **No mocking the WebRTC layer** in unit tests. Use `#if WEBRTC_AVAILABLE` guards and test against the mock/stub path.
4. **Data shaping tests** (`Data/CloudLibrary/`) are pure function tests — no async, no mocks needed.
5. **Performance tests** use the dedicated `Stratix-Perf` scheme.

### CI workflows (`.github/workflows/`)

| Workflow | What it validates |
|----------|------------------|
| `ci-app-build-and-smoke.yml` | Full app build + `AppSmokeTests` |
| `ci-packages.yml` | All SPM package tests |
| `ci-pr-fast-guards.yml` | Fast guard checks on PRs |
| `ci-shell-ui.yml` | Shell UI tests |
| `ci-shell-visual-regression.yml` | Visual regression checkpoints |
| `ci-runtime-safety.yml` | Runtime safety checks |
| `ci-release-and-validation.yml` | Release build + `Stratix-Validation` scheme |
| `ci-hardware-device.yml` | Hardware device integration |

---

## Execution Agreements

These are the rules all agents in this repo must follow. They are not optional.

### Structural agreements

1. **Do not create new files without justification.** The project already has a large source surface. New files must represent a genuinely new seam, not a preference for smaller files.
2. **Do not split files unless the split resolves a mixed-responsibility root.** Over-splitting is already an identified problem. Creating additional micro-shards makes it worse.
3. **Do not merge files unless the merge removes an artificial seam.** Merge only what is identified in the Floor/Execution contracts or a newly approved cleanup wave.
4. **Reference images must live in one canonical location.** `Apps/Stratix/Tools/shell-visual-regression/reference/` is the canonical location. Do not create a second reference image directory.
5. **Remove `.DS_Store` files** if encountered. Do not commit them.

### Code agreements

6. **Strict concurrency is always on.** Every new type must declare its isolation. If you add a class, decide: `@MainActor`, actor, or nonisolated with explicit Sendable conformance.
7. **`#if WEBRTC_AVAILABLE` guards are mandatory** in `Integration/WebRTC/`. Do not remove them or add WebRTC symbol references outside this directory.
8. **Do not add `@unchecked Sendable`** without a comment explaining why the type cannot conform normally and what invariant makes it safe.
9. **Action closures at UI boundaries** must be annotated `@MainActor` when the closure will be called from `@MainActor` context.
10. **`Task { @MainActor in ... }` is a smell at call sites** in view code. If you are wrapping a UI update in a detached Task inside a view, reconsider whether the caller should be async or whether the state object needs an actor boundary.

### Testing agreements

11. **New unit tests for pure-Swift logic** go in Swift Testing (`import Testing`).
12. **New unit tests for state machines** go under `StratixTests/` with a name matching the type under test (e.g., `CloudLibraryRouteStateTests.swift` for `CloudLibraryRouteState`).
13. **Do not add XCTest unit tests to targets that are migrating to Swift Testing.** Only add Swift Testing tests.
14. **Do not change the CI workflow files** without understanding what guards they enforce. If a workflow must change, document the reason in a comment in the file.

### Package agreements

15. **Packages do not import the app target.** The dependency direction is always app → package, never package → app.
16. **`StratixModels` has no package dependencies.** It must remain the leaf-level shared model package.
17. **New model types** that are shared across packages go in `StratixModels`, not in the app target.
18. **New network types** go in `XCloudAPI`, not in the app target.
19. **New diagnostic/metrics types** go in `DiagnosticsKit`, not in the app target.

---

## Modernization Contracts

For modernization and structural work, the canonical copied contract text lives in `Docs/Stratix_Modernization_Contracts.md`.

Use these documents together:
- `Docs/Stratix_Modernization_Contracts.md` — canonical AGENTS copy of the Floor Contract and Execution Contract
- `Docs/Stratix_Modernization_Plan.md` — execution inventory, phase order, and completion gates
- `Docs/Stratix_Monolith_Breakdown.md` — per-file move, merge, dissolve, and rename details
- `Docs/Stratix_File_Matrix.md` — whole-tree classification and destination tracking

When there is tension between them:
- The modernization plan remains the execution-order and approval-gate source of truth
- The contracts document remains the canonical AGENTS reference for contract language
- The monolith breakdown and file matrix are required cross-check artifacts for structural intent

---

## Audit Artifacts

A complete per-file structural audit was completed for the repo and the audit artifacts live at the repo root:

| Artifact | Purpose |
|----------|---------|
| `Stratix_Modernization_Contracts.md` | Canonical copied reference for the modernization Floor Contract and Execution Contract, including appended contract rules |
| `Stratix_File_Matrix.md` | Universal source-of-truth table — one row per file, recording classification, floor/execution status, current ownership, and post-refactor destination |
| `Stratix_Monolith_Breakdown.md` | Surgery manual for every file being split, merged, dissolved, or renamed — per-entity move plans, import deltas, call sites, and validation steps |
| `Stratix_Modernization_Plan.md` | Ordered execution program — phased from hygiene baseline through full modernization, with explicit prerequisites and validation criteria |

These files are the authoritative reference for any structural work in this repo. Read them before making decisions about file structure.

---

## Subdirectory Overrides

Each subdirectory listed below has its own `AGENTS.override.md` with specific context, rules, and working notes:

- `Apps/Stratix/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/App/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Auth/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Consoles/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Data/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Features/CloudLibrary/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Features/Guide/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Features/Streaming/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Integration/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Integration/WebRTC/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Shared/AGENTS.override.md`
- `Apps/Stratix/Sources/Stratix/Shell/AGENTS.override.md`
- `Packages/AGENTS.override.md`
- `Packages/DiagnosticsKit/AGENTS.override.md`
- `Packages/StratixCore/AGENTS.override.md`
- `Packages/StratixModels/AGENTS.override.md`
- `Packages/InputBridge/AGENTS.override.md`
- `Packages/StreamingCore/AGENTS.override.md`
- `Packages/VideoRenderingKit/AGENTS.override.md`
- `Packages/XCloudAPI/AGENTS.override.md`
- `Tools/AGENTS.override.md`
- `.github/AGENTS.override.md`

---
> Source: [nafields/stratix](https://github.com/nafields/stratix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
