## phosphor

> Phosphor is a Kotlin Multiplatform rendering library that translates cognitive state into visible light — ASCII luminance, color ramps, particle physics, and 3D waveform surfaces. Read [SOUL.md](SOUL.md) for project philosophy and values.

# AGENTS.md

Phosphor is a Kotlin Multiplatform rendering library that translates cognitive state into visible light — ASCII luminance, color ramps, particle physics, and 3D waveform surfaces. Read [SOUL.md](SOUL.md) for project philosophy and values.

## Development Commands

### Build & Test
```bash
./gradlew build                        # Build the project
./gradlew allTests                     # Run all tests
./gradlew jvmTest                      # Run JVM-specific tests (fastest, primary gate)
./gradlew iosSimulatorArm64Test        # Run iOS tests (Apple Silicon simulator)
./gradlew wasmJsBrowserTest            # Run WASM browser tests
```

### Code Quality
```bash
./gradlew ktlintFormat                 # Auto-format code
./gradlew ktlintCheck                  # Check formatting
./gradlew dokkaHtml                    # Generate API documentation
```

### Demo
```bash
./gradlew :phosphor-demo:run           # Run the desktop demo application
./gradlew :phosphor-demo-cli:installDist && ./phosphor-demo-cli/phosphor-demo  # Run the terminal demo
```

### Publishing
```bash
./gradlew publishToMavenLocal          # Publish to local Maven for integration testing
```

## Architecture at a Glance

Phosphor is a layered rendering pipeline: cognitive signals flow in, visible characters flow out. Each layer depends only on the layer below it — signal code never imports from palette, field code never imports from render.

| Layer | Location | Purpose |
|-------|----------|---------|
| Signal | `signal/` | CognitiveSignal data model — what the brain is doing, no knowledge of pixels or characters |
| Field | `field/` | ParticleField, SubstrateState, FlowLayer — animation physics at a fixed timestep |
| Palette | `palette/` | AsciiLuminancePalette, CognitiveColorRamp — mapping brightness and phase to characters and colors |
| Render | `render/` | AsciiCell, CellBuffer, 3D projection, waveform surface generation |
| Choreography | `choreography/` | CognitiveChoreographer — maps cognitive phases to particle behaviors and visual textures |
| Emitter | `emitter/` | EmitterEffect sealed hierarchy — transient visual events (SparkBurst, HeightPulse, Turbulence, ColorWash) |
| Bridge | `bridge/` | Adapters connecting live runtime state to the animation field |

### Rendering Surface Adapters (separate modules)

| Module | Purpose |
|--------|---------|
| `phosphor-mosaic/` | Mosaic (Compose-for-terminal) adapter — AnnotatedString output |
| `phosphor-compose/` | Compose Multiplatform Canvas adapter — DrawScope rendering |
| `phosphor-ansi/` | Raw ANSI escape code output for non-Compose terminals |

## Key Paths

| Module | Purpose |
|--------|---------|
| `phosphor-core/src/commonMain/` | Shared rendering pipeline — signals, fields, palettes, cells |
| `phosphor-core/src/commonTest/` | Unit tests for palette mapping, projection math, particle physics |
| `phosphor-mosaic/src/jvmMain/` | Terminal rendering via Mosaic AnnotatedString |
| `phosphor-compose/src/commonMain/` | Compose Canvas rendering via DrawScope |
| `phosphor-demo/` | Desktop demo app (Compose Desktop) |
| `phosphor-demo-cli/` | Terminal demo app |

## Before You Change Anything

- Read the file you're modifying. Read its tests.
- Understand the layer boundary you're in. The pipeline flows strictly downward: Signal → Field → Palette → Render → Surface Adapter.
- If touching particle physics (`field/`), run the demo to visually verify — unit tests alone can't catch motion regressions.
- If adding a new EmitterEffect, follow the sealed class pattern in `emitter/EmitterEffect.kt`.

## Kotlin Conventions

- Sealed classes/interfaces for closed type hierarchies (EmitterEffect, CognitivePhase, RenderTarget)
- Data classes for immutable value types (AsciiCell, CognitiveColorRamp, Particle)
- `expect`/`actual` for platform-specific rendering only — core math stays in `commonMain`
- Pure functions wherever possible; animation state is the only mutable concern
- ktlint with IntelliJ IDEA code style (auto-format with `./gradlew ktlintFormat`)

## Rendering Pipeline Rules

- Every `AsciiCell` must be constructible from `luminance + normal + palette + colorRamp` — no shortcut paths that bypass the palette system
- Particle physics runs at a fixed timestep (`update(dt: Float)`) independent of framerate
- EmitterEffects are transient — they fire once and decay. Persistent visual states belong in `field/`
- Character ramps must be ordered darkest-to-brightest. Unit tests enforce this invariant
- Color ramps must have at least 5 stops for smooth interpolation
- 3D projection must be configurable (camera position, field of view) — never hardcode perspective
- CellBuffer is the universal interchange format between rendering logic and surface adapters

## Testing

- **Primary gate:** `./gradlew jvmTest` — run this after every change
- Test luminance mapping boundaries: `charForLuminance(0.0f)` returns space, `charForLuminance(1.0f)` returns the brightest character
- Test color ramp interpolation at 0.0, 0.5, and 1.0
- Test particle spawn/decay lifecycle to completion
- Test 3D projection against known camera→screen coordinate pairs
- Visual regression tests: render a known CellBuffer to string and compare against golden files in `testdata/`

## Safety Boundaries

- **Never** add UI framework dependencies to `phosphor-core` — it must remain pure Kotlin
- **Never** hardcode terminal dimensions — always accept width/height parameters
- **Never** assume ANSI color support — surface adapters handle capability detection
- **Never** commit binary test assets without approval
- **Always** run `./gradlew jvmTest` before considering work complete
- **Always** run `./gradlew ktlintFormat` before committing
- **Always** verify demo output visually when changing palette or projection code

## Permissions

- **File read:** any file in the repository
- **File write:** `src/`, `test/`, `docs/`, config files
- **Git:** branch creation, commits (with review)
- **Build/test:** full access to `./gradlew` commands
- **External:** no network requests — this is an offline rendering library

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/socket-link) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
