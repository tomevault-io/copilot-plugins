## immich-tv

> Immich TV is a Flutter app for browsing a self-hosted Immich library on large screens. The product is TV-first, read-only, and optimized for remote navigation, fullscreen viewing, and slideshow playback.

# AGENTS.md

## Project Summary

Immich TV is a Flutter app for browsing a self-hosted Immich library on large screens. The product is TV-first, read-only, and optimized for remote navigation, fullscreen viewing, and slideshow playback.

Current priorities in `docs/DEVELOPMENT_CHECKLIST.md` are performance, degraded-network resilience, broader platform validation, and Android TV packaging readiness.

## Stack

- Flutter
- Dart 3.11
- `flutter_bloc` for presentation state
- Dio for API access
- `flutter_secure_storage` plus `shared_preferences` for saved profiles
- `video_player` for fullscreen playback

## Repository Map

- `lib/main.dart`: app startup and image cache tuning
- `lib/app.dart`: dependency wiring, theme setup, repository injection
- `lib/src/app_shell.dart`: top-level stage switch between bootstrap, onboarding, profile picker, and home
- `lib/src/core`: shared models, errors, repository interfaces, network setup, logging, config, services
- `lib/src/features`: feature-specific UI, Cubits, and repository implementations
- `lib/src/shared`: design tokens, breakpoints, viewport helpers, shared widgets
- `lib/src/platform`: platform-aware persistence adapters
- `test/`: widget coverage and focused unit tests
- `docs/DEVELOPMENT_CHECKLIST.md`: roadmap and current execution slices

## Architecture Conventions

- Keep repository interfaces in `lib/src/core/repositories`.
- Put API-backed implementations under the owning feature in `lib/src/features/.../data`.
- Keep state in Cubits and immutable state classes under `lib/src/features/.../cubit`.
- Treat `lib/src/shared/presentation` as the design-token and reusable-widget layer.
- Prefer constructor injection for repositories and services so widget tests can keep using fakes.
- Follow the existing top-level flow:
  bootstrap -> profile picker or onboarding -> authenticated home.

## TV-First UI Rules

- Preserve remote-first behavior. Keyboard arrows, `enter`, `select`, and pointer interaction should all remain usable.
- Reuse focus primitives such as `TvFocusable`, `FocusableSurface`, and related shared widgets instead of building ad hoc focus logic.
- Keep focus visibility obvious on large screens. New actionable surfaces should expose a clear focused state.
- Avoid raw spacing, radius, and color values when shared tokens already exist in `lib/src/shared/presentation`.
- When adding or changing scrolling surfaces, make sure focused items stay visible and do not trap remote navigation.

## Networking And Persistence

- Create HTTP clients through `ImmichDioFactory`; do not scatter custom Dio configuration.
- Preserve `ServerConfig` and `ServerUrlNormalizer` behavior, especially `/api` handling and reverse-proxy subpaths.
- Do not persist raw passwords in plain storage. Profiles and metadata belong in `SharedPreferences`; secrets belong in secure storage through `PlatformProfileStorage`.
- Keep auth and media access behind repository interfaces so mock mode and tests continue to work.

## Testing Expectations

- Run `flutter analyze` before finishing code changes.
- Run `flutter test` when behavior changes or state/navigation logic is touched.
- For UI tests, follow the existing pattern:
  - set a TV-class viewport such as `1280x720`
  - set `devicePixelRatio = 1.0`
  - use fake repositories instead of live network access
  - cover remote-style navigation when focus behavior changes
- Add focused unit tests for pure helpers and normalization logic.

## Common Commands

```bash
flutter pub get
flutter analyze
flutter test
flutter test test/widget_test.dart
```

## Change Guidance

- Favor small, composable changes inside the existing feature boundaries.
- If a task affects onboarding, auth, saved profiles, or fullscreen playback, inspect tests first because these flows already have coverage patterns to reuse.
- If a task touches performance, consider both UI rebuild behavior and image cache/prefetch behavior. `main.dart` already constrains Flutter image cache size.
- Keep README and the development checklist aligned when user-facing capabilities or roadmap status materially change.

---
> Source: [WorkWithAfridi/immich-tv](https://github.com/WorkWithAfridi/immich-tv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
