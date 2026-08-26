## innertubex

> This repository contains the reusable Android/JVM KMP InnerTubeX client, cipher/EJS, and SABR/UMP implementation.

# InnerTubeX Library

This repository contains the reusable Android/JVM KMP InnerTubeX client, cipher/EJS, and SABR/UMP implementation.

## Boundaries

- Do not depend on Metrolist application modules, models, settings, logging, playback engines, or UI.
- Keep host integration behind library-owned interfaces such as `InnerTubeLogger` and `PlayerConfigRepository`.
- Never log cookies, auth values, PO tokens, signed media URLs, media titles, or account identifiers.
- Never attach credentials to a caller-provided URL without validating its scheme, host, and endpoint path.
- Bound all remote response bodies and protocol allocations before materializing them in memory.
- Keep Android and desktop JVM source-set implementations in parity.
- Do not add hardcoded locale fallbacks; callers or the system locale must provide locale values.

## API Compatibility

- Keep public API minimal. `api/desktop/innertubex.api` is a committed binary API contract, not a disposable build artifact.
- After an intentional public API change, run `./gradlew desktopApiDump`, review the generated API diff, and commit it with the source change.
- Do not edit the API baseline manually. If no public API change is intended, investigate any diff instead of accepting it.
- Run `./gradlew apiCheck` before review and release.

## Git Hooks

- The version-controlled `.githooks/pre-commit` runs `ktlintCheck` and `apiCheck` without modifying files.
- After cloning, enable it with `git config core.hooksPath .githooks`.
- Do not make the hook auto-format source, regenerate API baselines, or stage changes. Those changes require explicit review.

## Verification

```bash
./gradlew ktlintFormat
./gradlew allTests ktlintCheck apiCheck assemble publishToMavenLocal
```

---
> Source: [MetrolistGroup/innertubex](https://github.com/MetrolistGroup/innertubex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
