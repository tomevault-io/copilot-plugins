## critter-vision

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

Gradle requires Java 21 from Android Studio's bundled JBR:

```bash
# Set for all commands (Git Bash / CLI)
export JAVA_HOME="C:/Program Files/Android/Android Studio/jbr"

# Build debug APK
./gradlew assembleDebug --no-daemon

# Run all unit tests
./gradlew test --no-daemon

# Run a single test class
./gradlew testDebugUnitTest --tests "net.raccooncode.crittervision.VisionColorFilterTest" --no-daemon
```

The `gradle-daemon-jvm.properties` enforces `toolchainVersion=21`. Uses Groovy Gradle (`build.gradle`), not Kotlin DSL.

## Architecture

Single-activity Android camera app that applies real-time animal vision color transformations via GPU shaders.

### Data flow

```
Camera → CameraX Preview UseCase
  → AnimalVisionEffect (CameraEffect wrapper)
    → AnimalVisionProcessor (SurfaceProcessor: EGL context + GLSL fragment shader)
      → OES texture input → mat3 color transform → PreviewView output
```

### Key files

- **`AnimalVisionProcessor.kt`** — OpenGL ES 2.0 SurfaceProcessor. Manages EGL context, compiles GLSL shaders, renders on a dedicated `HandlerThread`. The fragment shader pipeline: sRGB linearization → 3x3 color matrix → sRGB encoding → vec3 color offset → optional saturation boost → optional UV proxy → intensity blend.
- **`AnimalVisionEffect.kt`** — Thin CameraEffect wrapper passing the processor to CameraX.
- **`VisionColorFilter.kt`** — `VisionMode` enum (13 modes: Human, Dog, Cat, Bird, Eagle, Horse, Mantis Shrimp, Reindeer, Cuttlefish, Pit Viper + 3 debug) + `VisionParams` data class holding 3x3 matrix (row-major, linear RGB), 3-element colorOffset (sRGB space), saturation boost, and UV proxy weight. Matrices transposed to column-major for GLSL uniforms.
- **`MainActivity.kt`** — Sets up PreviewView, binds CameraX with `UseCaseGroup.addEffect()`, creates vision buttons programmatically in two rows: scrollable `HorizontalScrollView` for 9 animal modes, fixed equal-width `LinearLayout` for Human + 3 debug channels. Filter changes just update `@Volatile` fields on the processor — no shader recompilation.

### Threading model

- GL operations run on a dedicated `HandlerThread` ("AnimalVisionGL")
- Vision parameters (`colorMatrix`, `colorOffset`, `saturationBoost`, `uvProxyWeight`, `intensity`) are `@Volatile` for lock-free reads from the GL thread
- CameraX callbacks (`onInputSurface`, `onOutputSurface`) post work to the GL handler

## Color Science

All matrices operate on **linear RGB** (sRGB linearized in shader). Row sums ≈ 1.0 to preserve luminance.

- **Dog**: Machado et al. 2009 deuteranopia severity=1.0 — peer-reviewed, used by Chrome/Firefox accessibility tools
- **Cat**: Viénot/Brettel method at S=450nm L=554nm + 0.625 desaturation blend — no published cat-specific matrix exists; the desaturation factor models low cone density (25:1 rod:cone ratio)
- **Bird**: Heuristic spectral shift from Hart 2001 cone data + saturation ×1.35 (oil droplet narrowing) + UV proxy (true tetrachromacy is unrepresentable in 3-channel RGB)
- **Eagle**: Tetrachromatic + 4-8× acuity; enhanced color separation with sRGB brightness offset
- **Horse**: Dichromatic (S=428nm L=539nm); red/green collapse to muddy yellow
- **Mantis Shrimp**: 12-16 receptor hypersaturation (×1.80) + UV proxy (0.40)
- **Reindeer**: UV-sensitive; blue channel offset for snow/trail UV glow
- **Cuttlefish**: Polarization-sensitive; subtle saturation boost (×1.15) + brightness offset
- **Pit Viper**: Infrared pit organs; luminance → thermal red mapping (G/B zeroed)

## Project Config

- Package: `net.raccooncode.crittervision`, Min SDK 26, Target SDK 34
- CameraX 1.4.0 (camera-core, camera-camera2, camera-lifecycle, camera-view, camera-effects)
- AGP 9.0 with built-in Kotlin (no explicit Kotlin plugin — managed by AGP), AndroidX, XML layouts (no Compose)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shimmy0530)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Shimmy0530)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
