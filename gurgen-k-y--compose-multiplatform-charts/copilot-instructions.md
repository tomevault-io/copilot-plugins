## compose-multiplatform-charts

> This repository provides Canvas-based charts for Compose Multiplatform. The public Kotlin namespace is `io.github.gurgenky.charts`. The `charts` module is the only published library. `example` contains shared gallery UI and the desktop/Wasm launchers; `androidApp` packages the Android gallery.

# Compose Multiplatform Charts

## Project

This repository provides Canvas-based charts for Compose Multiplatform. The public Kotlin namespace is `io.github.gurgenky.charts`. The `charts` module is the only published library. `example` contains shared gallery UI and the desktop/Wasm launchers; `androidApp` packages the Android gallery.

## Commands

- `./gradlew check` — run all available checks.
- `./gradlew :charts:desktopTest` — run portable library tests.
- `./gradlew :example:wasmJsBrowserDevelopmentRun` — run the web gallery.
- `./gradlew :example:wasmJsBrowserDistribution :charts:dokkaGenerate` — build the Pages inputs.
- `./gradlew :charts:publishToMavenLocal` — validate local publication.

## Engineering rules

- Put shared behavior in `commonMain`; add platform code only when a Compose API requires it.
- Keep public models immutable and document every public declaration.
- Preserve source compatibility. Add deprecated forwarding APIs and migration text before removing public APIs.
- Keep Canvas drawing deterministic. Cache paths and brushes, clip to the plot viewport, and cull invisible data.
- Do not mutate Compose state from a drawing callback.
- Hoist caller-observable interaction state and key remembered state with all captured inputs.
- Validate non-finite values, invalid ranges, and mismatched series dimensions at API boundaries.
- Every behavior change needs focused tests and a gallery example.

## Git and releases

- Do not push, tag, or publish unless the user explicitly requests it.
- Never commit credentials, signing keys, generated build output, or local properties.
- Before a release, run tests, API validation, documentation generation, and local Maven publication.
- Maven Central publishing must run only from a version tag through the protected release environment.
- Publish only `:charts`; the gallery modules must never apply a publishing plugin.

Read matching files in `.claude/rules/` for path-specific requirements and use project skills for repeatable chart, upgrade, validation, and release workflows.

---
> Source: [gurgen-k-y/compose-multiplatform-charts](https://github.com/gurgen-k-y/compose-multiplatform-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
