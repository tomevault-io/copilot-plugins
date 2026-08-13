## safe-text

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Safe Text (`br.com.hoptech.safetext`) — a single-module Android app that encrypts/decrypts short text messages locally with a user-chosen passkey, so ciphertext can be pasted into any messaging app. No network, no accounts, no permissions declared in the manifest.

## Commands

```bash
./gradlew assembleDebug                      # build debug APK
./gradlew installDebug                       # build + install on connected device
./gradlew test                               # JVM unit tests (all variants)
./gradlew testDebugUnitTest                  # JVM unit tests, debug only
./gradlew testDebugUnitTest --tests "br.com.hoptech.safetext.crypto.CryptoEngineTest"
./gradlew testDebugUnitTest --tests "*CryptoEngineTest.round trip with unicode text"
./gradlew connectedDebugAndroidTest          # instrumented tests (device/emulator required)
./gradlew lint                               # Android Lint; report at app/build/reports/lint-results-debug.html
```

Backtick-quoted test method names work with `--tests` as long as the pattern is quoted in the shell.

There is no ktlint/detekt/spotless configured — Android Lint is the only static check.

## Build gotchas

- `app/google-services.json` is **gitignored** but required — the `com.google.gms.google-services` plugin fails the build without it. A fresh clone needs this file supplied out of band.
- The Firebase BOM is applied as a platform in `app/build.gradle.kts`, but no Firebase library is actually depended on; nothing in `src/` imports Firebase.
- Dependency versions live in `gradle/libs.versions.toml` (version catalog). AGP 9.0.0 / Kotlin 2.2.10 / minSdk 29 / target & compile SDK 36.
- Room uses **KSP**, not kapt.

## Architecture

Single Activity → single Compose screen → single ViewModel. No DI framework.

**Dependency wiring** is manual and lives in `SafeTextApplication`: it builds the Room database and `PasskeyRepository` in `onCreate`. `SafeTextViewModel` is an `AndroidViewModel` that reaches those via `application as SafeTextApplication`. Any new repository/service follows the same path — construct it in the Application, read it from the ViewModel. There is no ViewModel factory.

**Layers**

- `crypto/CryptoEngine` — stateless `object`. Format is `Base64(salt[16] ‖ iv[12] ‖ AES-GCM ciphertext+tag)`, key derived with PBKDF2WithHmacSHA256, 210 000 iterations, 256-bit key, 128-bit GCM tag. **This layout is the wire format**: changing any constant breaks decryption of every previously produced message, and there is no version byte to migrate on. Pure JVM (`javax.crypto`, `java.util.Base64`) so it is unit-testable without a device — `CryptoEngineTest` is the main test suite.
- `crypto/WordListProvider` — reads `res/raw/words_pt.txt` or `words_en.txt` (500 lines each) chosen from `Locale.getDefault().language`, and joins 3 random words with `-` for passkey suggestions.
- `data/` — Room: `PasskeyEntity` (`passkeys` table: id, label, passkey), `PasskeyDao` (`getAll()` returns a `Flow`, insert/delete are suspend), `PasskeyRepository`, `SafeTextDatabase` (**version 1, no migrations defined** — schema changes need a migration or the DB will crash on open). Passkeys are stored in **plaintext** in `safetext.db`; that is the current design, not an oversight to silently "fix".
- `ui/` — `SafeTextScreen` (Scaffold + TopAppBar + `PasskeySelector` + 2-page `HorizontalPager` with `TabRow`), `SafeTextViewModel` holding one immutable `UiState` in a `MutableStateFlow` updated via `update { copy(...) }`. Tab composables (`ui/encrypt`, `ui/decrypt`) are stateless — they take values and callbacks, and do their own clipboard work via `LocalClipboardManager`.

**Feedback message convention (important):** the ViewModel has no `Context` for strings, so `UiState.feedbackMessage` carries a **string key** (`"select_passkey_first"`, `"decryption_failed"`, …), and `SafeTextScreen` maps keys → `stringResource(...)` in a local `feedbackMessages` map before showing the snackbar. Adding a feedback case means adding the key in the ViewModel, the string in `values/strings.xml` **and** `values-pt-rBR/strings.xml`, and the entry in that map. An unmapped key is displayed raw.

**Localization:** English is the default; `values-pt-rBR` is a complete translation (24 strings each). Keep both in sync.

---
> Source: [hoptech/safe-text](https://github.com/hoptech/safe-text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
