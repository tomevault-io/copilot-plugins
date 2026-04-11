## mlkit-markdown-translator-android

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Project intent

Build a reusable Android Java library that translates Markdown with Google ML Kit while preserving Markdown structure.

Primary roadmap source: current docs under `docs/` (API, architecture, performance).

## Repository layout

- `library/` — core Android library module (main deliverable)
- `sample/` — Android app used for manual verification
- `docs/api.md` — public API reference
- `docs/architecture.md` — AST/token translation architecture

## Build, test, format

Use Gradle wrapper from repo root.

- Build sample app:
  - `./gradlew :sample:assembleDebug`
- Run library unit tests:
  - `./gradlew :library:testDebugUnitTest`
- Run formatting checks:
  - `./gradlew spotlessCheck`
- Auto-format Java + XML:
  - `./gradlew spotlessApply`

## Formatting rules

- Java formatting is enforced by Spotless + `googleJavaFormat` (AOSP style).
- XML formatting is enforced by Spotless (`eclipseWtp('xml')`).
- Run `spotlessApply` before finalizing changes.

## Implementation expectations

1. Keep library API small and stable.
2. Prefer AST/token-stream Markdown processing over regex-only logic.
3. Use regex fallback only for robustness/edge cases.
4. Preserve Markdown structure (headings, lists, quotes, code, links, spacing).
5. Keep Android/ML Kit integration separated from pure Java markdown logic where possible.

## Language/SDK constraints

- Java 17
- Android minSdk 24
- compileSdk 34

## Safety notes for changes

- Avoid introducing app-specific UI/business logic into `library/`.
- Keep sample UI behavior aligned with current UX requirements (toggle raw/rendered mode, model download/delete flow, etc.).
- Prefer additive/refactoring-safe changes with tests when modifying translation pipeline behavior.

## Validation checklist before handoff

1. `./gradlew spotlessCheck`
2. `./gradlew :library:testDebugUnitTest`
3. `./gradlew :sample:assembleDebug`

If device testing is requested:

- `adb install -r sample/build/outputs/apk/debug/sample-debug.apk`

## Release process (JitPack)

Use this flow to publish a new version on JitPack.

1. Update version references in docs/sample as needed (for example, `README.md` and `sample/build.gradle`).
2. Run validation from repo root:
   - `./gradlew spotlessCheck`
   - `./gradlew :library:testDebugUnitTest`
   - `./gradlew :sample:assembleDebug`
3. Ensure JitPack publishing config is present:
   - `library/build.gradle` includes `id 'maven-publish'`
   - release publication exists (`from components.release`, with sources jar)
   - `jitpack.yml` runs `./gradlew clean :library:publishReleasePublicationToMavenLocal`
4. Commit release-related changes to `main` and push.
5. Create and push a new tag:
   - `git tag -a X.Y.Z -m "Release X.Y.Z"`
   - `git push origin X.Y.Z`
6. Trigger/verify JitPack build:
   - Open `https://jitpack.io/#godsarmy/mlkit-markdown-translator-android`
   - Select tag `X.Y.Z` and wait for success
   - Optional log URL: `https://jitpack.io/com/github/godsarmy/mlkit-markdown-translator-android/X.Y.Z/build.log`

Notes:

- Prefer creating a new tag (for example `0.8.2`) instead of reusing/moving an existing tag.
- JitPack tags are effectively immutable for consumers; retagging can cause confusion/caching issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/godsarmy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/godsarmy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
