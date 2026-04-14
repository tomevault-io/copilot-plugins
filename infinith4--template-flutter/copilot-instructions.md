## template-flutter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flutter Music Bookmarker is a cross-platform mobile application for music playback with bookmark and group management features. Built with Flutter, it allows users to organize songs into groups, create time-specific bookmarks, and manage playlists with a Material Design 3 interface.

## Build Commands

All commands should be run from the `flutter-music-bookmarker` directory.

### Setup
```bash
cd flutter-music-bookmarker
flutter pub get
```

### Building
```bash
# Debug APK
flutter build apk --debug

# Release APK
flutter build apk --release
```

### Running
```bash
# Launch on connected device/emulator
flutter run

# Specific device
flutter devices
flutter run -d <device-id>
```

### Testing
```bash
flutter test
```

### Code Analysis
```bash
flutter analyze
flutter analyze --no-fatal-infos
```

### Cleaning
```bash
flutter clean
flutter pub get
```

## Architecture

### Layered Structure

``` 
flutter_music_bookmarker/
├── docs/ # Documents
├── lib/
   ├── main.dart                    # Entry point with MultiProvider setup
   ├── screens/
   │   └── home_screen.dart        # Main UI with DraggableScrollableSheet
   ├── providers/
   │   └── music_provider.dart     # State management (ChangeNotifier)
   ├── services/
   │   └── audio_player_service.dart  # Audio playback (Singleton)
   ├── database/
   │   └── database_helper.dart    # SQLite operations
   ├── models/
   │   ├── song.dart               # Song data model
   │   ├── group.dart              # Group data model
   │   ├── bookmark.dart           # Bookmark data model
   │   └── song_group.dart         # Many-to-many relationship
   └── widgets/
      ├── music_player_widget.dart   # Player UI component
      └── song_list_widget.dart      # Playlist UI component
```

### Core Patterns

**State Management**: Provider pattern with `ChangeNotifier`
- `MusicProvider` manages songs, groups, bookmarks, and loading states
- Wraps business logic and database operations
- Notifies listeners on state changes

**Database Access**: Repository-like pattern via `DatabaseHelper`
- Singleton SQLite connection
- CRUD operations with foreign key constraints and cascade deletion
- Four tables: songs, groups, bookmarks, song_groups

**Audio Service**: Singleton pattern with stream-based reactivity
- Wraps `just_audio` AudioPlayer
- Emits position, duration, and player state streams
- UI updates via StreamBuilder patterns

**Data Models**:
- **Song**: id, title, artist, filePath, duration, albumArt
- **Group**: id, name, createdAt (default "All Songs" group on init)
- **Bookmark**: id, songId, position, createdAt, note
- **SongGroup**: Junction table for many-to-many song-group relationships

### Key Dependencies

- **Audio**: just_audio, audio_service, on_audio_query
- **Database**: sqflite
- **State**: provider
- **Storage**: shared_preferences, path_provider
- **Permissions**: permission_handler
- **UI**: Material Design 3, intl (date formatting)

## Configuration

**Package**: `com.example.flutter_music_bookmarker`
**Min SDK**: 24 (Android 7.0)
**Target/Compile SDK**: 35 (Android 15)
**Java/Kotlin**: JDK 21
**Dart SDK**: ^3.10.0
**Flutter SDK**: 3.38.1 (Stable)
**Gradle**: 8.14

### Customization

- Package name: Change `applicationId` in `android/app/build.gradle.kts`
- App name: Modify `android:label` in `android/app/src/main/AndroidManifest.xml`
- Version: Update `version` in `pubspec.yaml`

## Release Process

### Initial Setup
```bash
# Generate keystore
keytool -genkey -v -keystore keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias myapp

# Create android/key.properties
storePassword=<password>
keyPassword=<password>
keyAlias=myapp
storeFile=keystore.jks
```

### Creating a Release

1. Update version in `pubspec.yaml`:
   ```yaml
   version: 1.1.0+2  # Increment version and build number
   ```

2. Create and push tag (use `flutter-v` prefix):
   ```bash
   git add pubspec.yaml
   git commit -m "Bump version to 1.1.0"
   git tag flutter-v1.1.0
   git push origin main --tags
   ```

3. GitHub Actions automatically:
   - Builds signed release APK
   - Creates GitHub release
   - Attaches APK to release

### GitHub Secrets Required

For CI/CD release builds:
- `FLUTTER_KEYSTORE_BASE64`: Base64-encoded keystore.jks
- `FLUTTER_KEYSTORE_PASSWORD`: Keystore password
- `FLUTTER_KEY_ALIAS`: Key alias
- `FLUTTER_KEY_PASSWORD`: Key password

## GitHub Actions Workflows

**flutter-build.yml** executes:

1. **build**: Runs on every push to flutter-music-bookmarker path
   - flutter pub get, analyze, test
   - Debug APK build
   - Uploads APK as artifact

2. **release**: Runs only on flutter-v* tags
   - Decodes keystore from secrets
   - Creates key.properties
   - Builds signed release APK
   - Creates GitHub release

**Triggers**: Push to main/develop/claude/** branches affecting flutter-music-bookmarker/**, PRs, manual dispatch

## Testing

### Running Tests
```bash
cd flutter-music-bookmarker
flutter test
```

### Emulator Testing
```bash
# List available emulators
emulator -list-avds

# Launch emulator
emulator -avd 'Pixel 9a'

# Run app
flutter run
```

## Important File Locations

- **Source code**: `flutter-music-bookmarker/lib/`
- **Android config**: `flutter-music-bookmarker/android/app/build.gradle.kts`
- **Dependencies**: `flutter-music-bookmarker/pubspec.yaml`
- **Lint config**: `flutter-music-bookmarker/analysis_options.yaml`
- **Gradle properties**: `flutter-music-bookmarker/android/gradle.properties`
- **Design docs**: `flutter-music-bookmarker/docs/`
- **CI/CD**: `.github/workflows/flutter-build.yml`

## Troubleshooting

### Dependency Errors
```bash
flutter clean && flutter pub get
```

### Build Errors
```bash
cd flutter-music-bookmarker/android
./gradlew clean
cd ..
flutter build apk
```

### Permission Issues

App requires audio/storage permissions. Grant manually if needed:
Settings → Apps → Music Bookmarker → Permissions → Storage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/infinith4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
