## plagued-pixel-dungeon

> Checklist for an AI coding agent (what I'll do first):

# AGENTS: How to be productive in this codebase

Checklist for an AI coding agent (what I'll do first):
- Read the module layout and settings.gradle to understand high-level boundaries
- Open `core` to find game logic, `desktop`/`android`/`ios` for platform glue
- Inspect `SPDSettings`, `Assets`, and `ShatteredPixelDungeon` for global conventions
- Locate services (`services/*`) to see pluggable network integration patterns

Quick architecture summary
- Multi-module Gradle project (see `settings.gradle`): core game code lives in `:SPD-classes` and `:core`.
- Platform modules implement runtime/platform glue: `:desktop`, `:android`, `:ios`. They provide PlatformSupport implementations (e.g. `desktop/DesktopPlatformSupport`, `ios/IOSPlatformSupport`) and launcher classes (e.g. `desktop/DesktopLauncher`).
- Services are pluggable under `services` with categories (updates, news). Concrete implementations are provided per module and selected at runtime (`UpdateImpl`, `NewsImpl`). Example: `services/updates/githubUpdates` implements GitHub-based checks (see `GitHubUpdates.java`).
- Assets and messages are centralized: `core/src/main/java/.../Assets.java` lists resource keys; text bundles live under `core/src/main/resources`-style bundles referenced in `Assets.Messages` (look in `core/src/main/java/.../messages/`).

Important code patterns and conventions
- Scene-based UI: the app uses a PixelScene/GameScene hierarchy. Scene switching helpers are on `ShatteredPixelDungeon` (e.g. `switchNoFade`, `seamlessResetScene`) — change behavior here to affect all scene transitions.
- Settings and preferences: centralized in `SPDSettings` (singleton-like static API). Use `SPDSettings` for any persistent configuration (display, audio, language, connectivity). Example: `SPDSettings.music()` controls `Music.INSTANCE` in `ShatteredPixelDungeon.create()`.
- Assets are referenced by key strings in `Assets.java`. If you add/rename an asset, update `Assets.java` and place the file in the platform assets folder (Android: `android/assets/`).
- Bundle-based serialization: the project uses `com.watabou.utils.Bundle` extensively for save files and remote data (see `GitHubUpdates` reading JSON via `Bundle.read`). Also note aliasing for backwards compat in `ShatteredPixelDungeon` constructor using `Bundle.addAlias(...)`.
- Platform selection at runtime: platform-specific services are discovered via `UpdateImpl.supportsUpdates()` / `NewsImpl.supportsNews()` and assigned to `Updates.service` / `News.service` in launchers (see `DesktopLauncher`). Implementations live under `services/*` or the platform module.
- Versioning: version codes and compatibility constants are defined on `ShatteredPixelDungeon` and `build.gradle` (`appVersionCode`, `appVersionName`). Savegame compatibility and guardrails are implemented using these constants.

Developer workflows (commands & notes)
- Gradle wrapper (Windows PowerShell): use the included `gradlew.bat` from project root.
  - Run Desktop debug (fast iterate): .\gradlew.bat desktop:debug
  - Build Desktop release JAR: .\gradlew.bat desktop:release
  - Build Android (assemble/debug or assemble/release): .\gradlew.bat :android:assembleDebug  or  .\gradlew.bat :android:assembleRelease
  - Build specific service module if you change it: .\gradlew.bat :services:updates:githubUpdates:build
- Recommended for rapid iteration: edit platform-independent code in `core` and run the `desktop:debug` target (faster feedback than building Android every change).
- Runtime entrypoints for debugging: `desktop/DesktopLauncher.main` and `core/ShatteredPixelDungeon` (constructor + `create()`) — these are good places to attach logs or temporary breakpoints.

How the services plug in (example)
- `UpdateService`/`News` are abstract/utility entrypoints. Platform modules call concrete implementations in launcher code (DesktopLauncher sets `Updates.service = UpdateImpl.getUpdateService()` if supported).
- To add/modify an update source: edit `services/updates/githubUpdates/src/.../GitHubUpdates.java` (parses GitHub releases and expects a release body with a line like "internal version number: #").
- To change news feed source: edit `services/news/shatteredNews` or `services/news/debugNews` (see `docs/recommended-changes.md` for pointers used by the project).

Project-specific gotchas and conventions
- This repository intentionally does not accept pull requests (see root `README.md`). Treat the tree as a reference fork; distribution/licensing is GPLv3 — if you plan to publish changes, read `docs/*` (Android distribution warning).
- Many UI/layout decisions are density/size-conditional. `SPDSettings.interfaceSize()` uses device detection to choose mobile vs full UI; changing UI logic often requires testing both desktop and mobile resolutions.
- Assets vs message bundles: message keys point to resource bundles (see `Assets.Messages` constants). Don't change the string constants unless you also move the resource file.
- Exception handling on Desktop: `DesktopLauncher` installs a global uncaught exception handler that condenses and displays errors. For reproducible crashes, the handler sanitizes traces — check `Game.reportException` for crash reporting behavior.
- Preferences: platform-specific code sets `SPDSettings.set(...)` with a platform preference implementation (example `Lwjgl3Preferences` in `DesktopLauncher`). Changing preference storage requires updating platform modules.

Where to look next (high-value files/directories)
- High-level: `settings.gradle`, `build.gradle` (root)
- Core game logic: `core/src/main/java/com/shatteredpixel/shatteredpixeldungeon/` (look at `scenes/`, `items/`, `actors/`, `levels/`)
- Platform glue: `desktop/src/main/java/.../DesktopLauncher.java`, `desktop/DesktopPlatformSupport.java`, `ios/src/main/java/.../IOSPlatformSupport.java`, `android/src/main/...`
- Services: `services/updates/*`, `services/news/*` (see `GitHubUpdates.java` for parsing examples)
- Config & assets: `core/.../SPDSettings.java`, `core/.../Assets.java`, `docs/getting-started-*.md` for build notes

Short sample edits an agent might make
- Add a new update checker: implement `services/updates/myService/MyUpdates.java` following `UpdateService` API, register it in the platform launcher (e.g. `DesktopLauncher`) by adding `if (MyUpdateImpl.supportsUpdates()) Updates.service = MyUpdateImpl.getUpdateService();`.
- Add localized strings: add resource bundle under `core/.../messages/` and add constant in `Assets.Messages`.

If you need more specific guidance, tell me which file or feature you want to modify and I will open the relevant files and provide a concrete edit plan.

---
> Source: [triiifish/Plagued-Pixel-Dungeon](https://github.com/triiifish/Plagued-Pixel-Dungeon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
