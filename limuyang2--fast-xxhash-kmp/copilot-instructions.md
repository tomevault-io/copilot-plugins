## fast-xxhash-kmp

> This is a Kotlin Multiplatform xxHash library with demo apps. The core API lives in `lib/src/commonMain/kotlin/io/github/limuyang2/xxhash/lib`, with platform implementations under `androidMain`, `iosMain`, `jvmMain`, `webMain`, and tests under matching `*Test` source sets. Android JNI and shared C sources are in `lib_android_native/src/main/cpp`. Demo entry points are split across `androidApp`, `iosApp`, `webApp`, and shared Compose UI in `commonApp`.

# Repository Guidelines

## Project Structure & Module Organization

This is a Kotlin Multiplatform xxHash library with demo apps. The core API lives in `lib/src/commonMain/kotlin/io/github/limuyang2/xxhash/lib`, with platform implementations under `androidMain`, `iosMain`, `jvmMain`, `webMain`, and tests under matching `*Test` source sets. Android JNI and shared C sources are in `lib_android_native/src/main/cpp`. Demo entry points are split across `androidApp`, `iosApp`, `webApp`, and shared Compose UI in `commonApp`.

## Build, Test, and Development Commands

Use the Gradle wrapper from the repository root:

```bash
./gradlew :lib:compileKotlinJs
./gradlew :lib:compileKotlinWasmJs
./gradlew :lib:compileKotlinMetadata
./gradlew :androidApp:installDebug
./gradlew :webApp:jsBrowserDevelopmentWebpack
```

Run focused tests with:

```bash
./gradlew :lib:jvmTest
./gradlew :lib:jsNodeTest
./gradlew :lib:wasmJsNodeTest
./gradlew :lib:iosSimulatorArm64Test
```

Android device tests require an emulator or device:

```bash
./gradlew :lib:connectedAndroidTest
```

## Coding Style & Naming Conventions

Kotlin uses the official style (`kotlin.code.style=official`) with 4-space indentation. Keep package names under `io.github.limuyang2.xxhash`. Public hash APIs use concise `xxh*` naming, matching existing functions such as `xxh32`, `xxh64`, and `xxh3As128`. Platform-specific files should use suffixes already present in the project, for example `XXHash.android.kt`, `XXHash.ios.kt`, or `XXHash.web.kt`. Keep C/JNI changes localized to `lib_android_native` unless the public Kotlin API changes.

## Testing Guidelines

Use `kotlin.test` for multiplatform library tests and JUnit/AndroidX for Android tests. Add new test cases beside the affected platform source set, such as `lib/src/jvmTest`, `lib/src/webTest`, `lib/src/iosSimulatorArm64Test`, or `lib/src/androidDeviceTest`. Prefer deterministic vectors and verify signed `Long` results carefully against unsigned xxHash reference values.

## Commit & Pull Request Guidelines

Recent commits use short subjects and conventional prefixes when helpful, for example `fix(xxhash): ...`, `docs(readme): ...`, and `chore(build): ...`. Keep commits scoped to one behavior or module. Pull requests should describe the change, list tested Gradle tasks, mention affected platforms, and link issues when available. Include screenshots only for demo UI changes.

## Agent-Specific Instructions

When answering questions about libraries, frameworks, SDKs, APIs, CLI tools, or cloud services, use Context7 MCP first: resolve the library ID, query current docs, then answer from those docs. Do not use Context7 for general refactoring, business logic debugging, code review, or scripts written from scratch.

---
> Source: [limuyang2/fast-xxhash-kmp](https://github.com/limuyang2/fast-xxhash-kmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
