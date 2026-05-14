## flutter-confetti-engine

> A Flutter package that unifies celebration particle animation, haptic feedback, and optional sound

# flutter_confetti_engine — Project Rules for Claude

## What This Package Is
A Flutter package that unifies celebration particle animation, haptic feedback, and optional sound
in a single API call. Full specification: `doc/flutter_confetti_engine_spec.docx`.

---

## Architecture Rules

### Layer boundaries (never cross them)
```
Public API  →  Preset Layer  →  Physics Core
```
- `confetti_engine.dart` and `confetti_widget.dart` are the **only** files consumers touch.
- `presets.dart` may import from `particle.dart`, `particle_shape.dart`, `flutter_confetti_physics.dart`, `flutter_confetti_constants.dart`, `tick_confetti_physics.dart`, and `tick_confetti_spawn_options.dart` only.
- `confetti_painter.dart` may import from `particle.dart`, `particle_shape.dart`, `flutter_confetti_physics.dart`, and `tick_confetti_physics.dart` only.
- `audio_player_wrapper.dart` is imported exclusively by `confetti_widget.dart`.
- **No circular imports. Ever.**

### File ownership (one responsibility per file)
| File | Owns |
|------|------|
| `particle_shape.dart` | The `ParticleShape` enum (13 values) only |
| `confetti_customization.dart` | `ConfettiCustomization` only |
| `particle.dart` | `Particle` data class + `update(double dt)` |
| `flutter_confetti_constants.dart` | `kFlutterConfettiDesiredSpeed` (= 60), preset integration constant |
| `flutter_confetti_physics.dart` | `FlutterConfettiPhysics` integrator + paper/star paths |
| `tick_confetti_spawn_options.dart` | `TickConfettiSpawnOptions` + default / warm palettes for tick mode |
| `tick_confetti_physics.dart` | `TickConfettiPhysics` — per-frame tick integrator (wobble, decay, quad corners) |
| `confetti_painter.dart` | `CustomPainter` — draws all [ParticleShape] variants |
| `presets.dart` | `Preset` enum + `PresetFactory` static methods |
| `confetti_showcase.dart` | `ConfettiShowcase` enum + `ShowcaseParticleFactory` (curated tick bursts) |
| `confetti_controller.dart` | `ConfettiController extends ChangeNotifier` |
| `confetti_widget.dart` | `ConfettiWidget` StatefulWidget + Ticker loop |
| `audio_player_wrapper.dart` | Nullable audioplayers wrapper with try/catch |
| `confetti_engine.dart` | `ConfettiEngine` static API + OverlayEntry management |
| `confetti_color_theme.dart` | `ConfettiColorTheme` + `ConfettiColorThemes` palettes |
| `confetti_type.dart` | `ConfettiType` enum |
| `confetti_style.dart` | `ConfettiStyle` + `ConfettiStyleShapes` |
| `confetti_animation.dart` | `AnimationConfetti` + `ConfettiAnimationMaps` |
| `confetti_density.dart` | `ConfettiDensity` + `ConfettiDensityScale` |
| `celebration_scene.dart` | `CelebrationScene` factories (`fromConfettiType`, `compose`, `themed`) |
| `celebration_message_options.dart` | `CelebrationMessageOptions` overlay text + duration |
| `flutter_confetti_engine.dart` | Barrel export — public surface only |

---

## Physics Rules
- **Legacy particles** (`Particle.flutterConfetti == null`): update formula is **exactly**:
  ```dart
  vy += gravity * dt;
  x  += vx * dt;
  y  += vy * dt;
  rotation += rotationSpeed * dt;
  currentLifetime += dt;
  opacity = (1.0 - currentLifetime / maxLifetime).clamp(0.0, 1.0);
  ```
- **Preset particles (default)** use integrated semantics via `FlutterConfettiPhysics`: quadratic drag, startup burst (first 5 frames), wind (first 25 frames), gravity vector `lerpDouble(0.1, 5, gravity01)`, `deltaTime * desiredSpeed` scaling (`desiredSpeed = 60`), tumble angles, and **Matrix4** path rendering.
- **Optional tick engine** when `ConfettiCustomization.useTickBasedPhysics` is true: `TickConfettiPhysics` — one [update] per frame, `velocity *= decay`, wobble/tilt, opacity `1 - progress`, and **Canvas** shapes (star path, scaled oval circle, quadrilateral square/triangle). Customization: [TickConfettiSpawnOptions] + `tickSpawnOptions` override.
- Never change legacy physics without updating `test/particle_test.dart`.
- `shouldRepaint` always returns `true` during animation; the Ticker stop handles idle.

## Rendering Rules
- All shapes are drawn **centered on (0, 0)** after `canvas.translate(x, y)`.
- Always wrap shape drawing in `canvas.save()` / `canvas.restore()`.
- `ParticleShape.emoji` uses `TextPainter` — center it at `(-width/2, -height/2)`.
- `ParticleShape.star` uses 10 alternating points (outer/inner radius at 40%).
- Remaining shapes (`pentagon`, `hexagon`, `ring`, `lightning`, `crescent`, `arrow`, and basic `circle`, `square`, `triangle`, `ribbon`) follow the same centering rule; `ring` uses stroke-only paint; `crescent` uses even-odd fill.

## Preset Rules
Factory-tuned preset styles:

| Preset | Count | Gravity | Haptic |
|--------|-------|---------|--------|
| nova | 120 | 500 | heavyImpact |
| cascade | 200 | 480 | heavyImpact |
| flare | 72 | 220 | mediumImpact |
| crossfire | 100 | 280 | mediumImpact |

- Never change these values without updating `test/preset_test.dart`.

## API Rules
- `ConfettiEngine.celebrate()` must remain a static one-liner — no required params beyond `context`.
- Optional `showcase:` ([ConfettiShowcase]) drives tick bursts instead of [preset] when non-null; pair [ConfettiShowcase.manualLaunchKill] with [ConfettiController.kill] for manual lifecycle demos.
- [ConfettiCustomization.particleBlendMode] applies to vector fills/strokes; [ConfettiEngine.celebrateInDialog] wraps the same [ConfettiWidget] in a transparent full-screen [Dialog].
- `enableHaptics` defaults to `true`, `enableSound` defaults to `false`.
- `ConfettiEngine` maintains a single static `_currentEntry`; calling `celebrate()` again dismisses any active one first.
- `ConfettiController.dispose()` must be called by the host — document this clearly.
- `ConfettiWidget` with `autoPlay: true` and no controller starts immediately after first layout.

## Sound Rules
- **All** `audioplayers` calls are inside `try/catch (_) {}` — sound never crashes the app.
- `AudioPlayerWrapper` is only instantiated when `enableSound: true` AND `soundAssetPath != null`.
- The wrapper exposes only `play(String assetPath)` and `dispose()`.

## Haptic Rules
- Haptics fire in `confetti_widget.dart` inside `_start()`, not inside `PresetFactory`.
- Snow preset fires NO haptic, even if `enableHaptics: true`.
- Use only: `HapticFeedback.heavyImpact()` and `HapticFeedback.mediumImpact()`.

## Dart & Flutter Style
- Dart SDK: `>=3.0.0 <4.0.0`, Flutter: `>=3.10.0`.
- Use `const` constructors wherever possible.
- No `dynamic` types anywhere in the public API surface.
- All public members must have dartdoc comments (`///`).
- Private helpers use `_camelCase`.
- Run `flutter analyze` before every commit — zero warnings allowed.
- Run `flutter test` — all tests must be green.

## What NOT To Do
- Do NOT use `Widget` per particle — CustomPainter only.
- Do NOT use native platform channels — Flutter APIs only.
- Do NOT expose `PresetFactory` or `Particle` in the barrel export.
- Do NOT make sound a required dependency — it must remain opt-in.
- Do NOT add new public API surface without updating the spec doc.
- Do NOT use `dart:io` in library code — must work on web.

---

## Project Structure
```
flutter_confetti_engine/
  CLAUDE.md                          ← this file
  pubspec.yaml
  analysis_options.yaml
  doc/
    flutter_confetti_engine_spec.docx  ← full specification
  lib/
    flutter_confetti_engine.dart       ← barrel export
    src/
      particle_shape.dart
      particle.dart
      presets.dart
      confetti_showcase.dart
      confetti_painter.dart
      confetti_controller.dart
      confetti_widget.dart
      audio_player_wrapper.dart
      confetti_engine.dart
  example/
    pubspec.yaml
    lib/main.dart
  test/
    particle_test.dart
    preset_test.dart
    controller_test.dart
```

## Build Order
Always implement in this order (respects dependency graph):
1. `particle_shape.dart`
2. `particle.dart`
3. `presets.dart`
4. `confetti_painter.dart`
5. `audio_player_wrapper.dart`
6. `confetti_controller.dart`
7. `confetti_widget.dart`
8. `confetti_engine.dart`
9. `flutter_confetti_engine.dart` (barrel)
10. Tests
11. Example app

---
> Source: [iuzairaslam/flutter_confetti_engine](https://github.com/iuzairaslam/flutter_confetti_engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
