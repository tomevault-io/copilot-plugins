## opencode-android-client

> 终端默认可能找不到 Java，导致 `./gradlew` 失败。使用 Android Studio 自带的 JDK：

# AGENTS.md - opencode_android_client

## Build Environment

终端默认可能找不到 Java，导致 `./gradlew` 失败。使用 Android Studio 自带的 JDK：

```bash
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"

# For integration tests (adb)
export PATH="$PATH:$HOME/Library/Android/sdk/platform-tools"
```

**持久化**：在 `~/.zshrc` 中加入上述 `JAVA_HOME` 和 `PATH` 两行，然后 `source ~/.zshrc`。

## Run / Module not found

若 Run 报 "Module not found"：File → Sync Project with Gradle Files；若仍失败，File → Invalidate Caches / Restart。Run 配置使用 module `opencode_client.app`（对应 settings.gradle.kts 的 rootProject.name + `:app`）。

## Test Commands

- Unit tests: `./gradlew testDebugUnitTest`
- Coverage report: `./gradlew koverHtmlReport` → `app/build/reports/kover/html/index.html`
- Integration tests: `./gradlew connectedDebugAndroidTest` (requires .env with OPENCODE_* credentials)

---
> Source: [grapeot/opencode_android_client](https://github.com/grapeot/opencode_android_client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
