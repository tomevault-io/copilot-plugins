## composenativewebview

> - `demo/`: Compose Desktop sample app (`demo/src/jvmMain/kotlin/...`).

# Repository Guidelines

## Project Structure & Module Organization

- `demo/`: Compose Desktop sample app (`demo/src/jvmMain/kotlin/...`).
- `wrywebview/`: native WebView core.
  - Rust crate: `wrywebview/Cargo.toml` with sources in `wrywebview/src/main/rust/` (wry + UniFFI).
  - JVM glue: `wrywebview/src/main/kotlin/` + `wrywebview/src/main/java/` (JNA/Skiko interop).
- `wrywebview-compose/`: Compose wrapper exposing `io.github.kdroidfilter.webview.*` (`WebView`, `WebViewState`, `WebViewNavigator`).
  - Shared API/types: `wrywebview-compose/src/commonMain/kotlin/...`.
  - Platform actuals: `.../src/jvmMain/` (Wry), `.../src/androidMain/` (Android WebView), `.../src/iosMain/` (WKWebView + cinterop in `.../src/nativeInterop/`).
- Generated/build outputs live under `*/build/` and `wrywebview/target/` (don’t edit or commit).

## Build, Test, and Development Commands

- `./gradlew build`: builds all modules (Kotlin + Rust via Gobley/UniFFI); requires a working Rust toolchain.
- `./gradlew :demo:run`: runs the desktop demo app.
- `./gradlew :wrywebview:build`: rebuilds the native core and refreshes generated bindings.
- `./gradlew :wrywebview-compose:compileDebugKotlinAndroid`: compiles the Android implementation (requires Android SDK).
- `./gradlew clean`: removes Gradle build outputs (useful when native artifacts get out of sync).

## Coding Style & Naming Conventions

- Kotlin/Compose: 4-space indentation, idiomatic Kotlin style, `camelCase` for values/functions, `PascalCase` for types and `@Composable` functions (e.g., `WebView`).
- Keep public API changes small and documented (README usage snippets should stay accurate).
- Rust: format with `cargo fmt` in `wrywebview/`; keep the `#[uniffi::export]` surface stable and cross-platform.

## Testing Guidelines

- Kotlin tests (when added) should live in `*/src/jvmTest/kotlin` (or `commonTest`) and run with `./gradlew test`.
- Rust tests (when added) can run via `cd wrywebview && cargo test`.

## Commit & Pull Request Guidelines

- Commit messages follow a simple imperative style (e.g., “Add …”, “Fix …”, “Refactor …”) and mention the affected module/API when helpful.
- PRs should include: a short rationale, steps to verify (`./gradlew :demo:run`), OS tested (Linux/macOS/Windows), and screenshots/GIFs for UI changes.

## Security & Configuration Tips

- The demo/app JVM needs `--enable-native-access=ALL-UNNAMED` (JNA); keep this in sync with `README.md`.
- Platform builds may require system deps (notably GTK/WebKit on Linux); call out any new requirements in the PR description.

---
> Source: [kdroidFilter/ComposeNativeWebview](https://github.com/kdroidFilter/ComposeNativeWebview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
