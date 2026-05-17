## audiooutputswitcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Android application that provides two Quick Settings tiles for audio control: one for switching audio output devices and one for opening the system volume panel.

## Build Commands

- **Build the project**: `./gradlew build`
- **Assemble debug APK**: `./gradlew assembleDebug`
- **Assemble release APK**: `./gradlew assembleRelease`
- **Install debug build on connected device**: `./gradlew installDebug`
- **Run unit tests**: `./gradlew test`
- **Run instrumented tests**: `./gradlew connectedAndroidTest`
- **Clean project**: `./gradlew clean`

## Architecture

### Core Components

- **AudioOutputTileService**: Tile for switching audio output devices
  - Location: `app/src/main/java/br/com/wasystems/audiooutputswitcher/AudioOutputTileService.kt`
  - Integrates with Android System UI to open the media output dialog
  - Uses PendingIntent to send broadcast to SystemUI's MediaOutputDialogReceiver
  - Falls back to volume panel / sound settings if broadcast is blocked

- **VolumePanelTileService**: Tile for opening the system volume panel
  - Location: `app/src/main/java/br/com/wasystems/audiooutputswitcher/VolumePanelTileService.kt`
  - Opens `Settings.Panel.ACTION_VOLUME` on tap (same fallback chain via `AudioOutputFallback`)
  - Solves the Nothing Phone use case where tiles pinned as home screen widgets can't receive long press events

- **TileServiceExt**: Shared extension function for both tile services
  - Location: `app/src/main/java/br/com/wasystems/audiooutputswitcher/TileServiceExt.kt`
  - `tryStartActivity(intent, tag)` wraps `startActivityAndCollapse` with Android 14+ API compatibility

### Key Configuration

- **Version**: 1.0.10 (versionCode 11)
- **Compile SDK**: 36 (Android 16)
- **Target SDK**: 36 (Android 16)
- **Min SDK**: 30 (Android 11+)
- **Java/Kotlin**: Version 17
- **Namespace**: `br.com.wasystems.audiooutputswitcher`
- **Build features**: Gradle configuration cache enabled, code minification and resource shrinking enabled for release builds
- **F-Droid compatibility**: `dependenciesInfo { includeInApk = false }` strips Google Play dependency metadata from the APK

### Manifest Configuration

The app registers two Quick Settings tile services, both with:
- Permission: `android.permission.BIND_QUICK_SETTINGS_TILE`
- Intent filter: `android.service.quicksettings.action.QS_TILE`
- `android:value="false"` in the TILE meta-data — tiles are inactive by default and must be manually added via the Quick Settings editor

Icons: `@drawable/ic_audio_output` (speaker) and `@drawable/ic_volume_panel` (tune/sliders)

### System Integration

The tile communicates with Android's SystemUI through:
- Action: `com.android.systemui.action.LAUNCH_SYSTEM_MEDIA_OUTPUT_DIALOG`
- Target: `com.android.systemui.media.dialog.MediaOutputDialogReceiver`

When the broadcast is blocked (e.g. on custom ROMs), `AudioOutputTileService` falls back in cascade:
1. `Settings.Panel.ACTION_VOLUME` — Volume Panel (includes output switcher on stock Android)
2. `Settings.ACTION_SOUND_SETTINGS` — Sound Settings page
3. Toast error message if all fallbacks fail

`VolumePanelTileService` uses the same fallback chain starting at step 1.

### Localization

Supports Portuguese (Brazil) localization in addition to English.

## Development Notes

- The app has no main Activity - it's purely a tile service
- Error handling includes logging and Toast messages for user feedback
- Uses modern Android development practices with Kotlin and AndroidX libraries
- Gradle configuration cache is enabled for improved build performance

---
> Source: [weslley75/AudioOutputSwitcher](https://github.com/weslley75/AudioOutputSwitcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
