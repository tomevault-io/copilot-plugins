## rdzwx-go

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rdzwx-go** is a hybrid mobile application for radiosonde (weather balloon) tracking and visualization built with Apache Cordova. The project consists of two main components:

- **rdzwx-go/**: Main Cordova application (HTML/JS frontend)
- **rdzwx-plugin/**: Custom native plugin (Kotlin for Android, Objective-C for iOS, Node.js for Electron)

## Common Development Commands

### Build Commands (from Makefile)
```bash
make run          # Build and deploy to Android device
make full         # Clean rebuild with plugin reinstall (use when plugin changes)
make plugin       # Reinstall plugin only
make el           # Build for Electron platform
make release      # Create signed release APK
```

### Core Cordova Commands
```bash
cordova build                    # Build debug version
cordova build --release         # Build release version  
cordova run android --device    # Deploy to Android device
cordova run ios --device        # Deploy to iOS device (requires Xcode on macOS)
cordova platform add android    # Add Android platform
cordova platform add ios        # Add iOS platform
cordova plugin add plugin-src/rdzwx-plugin/  # Add custom plugin
```

### Development Setup
```bash
# Initial setup after clone
cordova platform add android
cordova platform add ios        # iOS platform (requires macOS)
npm i jetifier && npx jetifier    # For AndroidX compatibility
```

### Android Emulator Commands
```bash
# List available AVDs
emulator -list-avds

# Run emulator with optimized settings (hardware GPU acceleration)
emulator -avd <avd_name> -gpu host -no-audio

# Run with snapshots for faster boot (recommended after initial setup)
emulator -avd <avd_name> -gpu host

# Note: KVM acceleration is automatically used on Linux if available
# CPU usage is high during boot (~300%) but drops to ~30% after boot completes
```

## Architecture

### Multi-Platform Hybrid Architecture
- **Frontend**: HTML5/JavaScript with Leaflet.js mapping in `rdzwx-go/www/`
- **Native Layer**: Custom Cordova plugin bridges JS to native functionality
- **Android Implementation**: Kotlin with Mapsforge offline mapping libraries
- **iOS Implementation**: Objective-C with Core Location, Network.framework, and NSNetServiceBrowser
- **Electron Implementation**: Node.js for desktop platform

### Key Technologies
- Apache Cordova v9.0.0+ for hybrid mobile framework
- Leaflet.js for interactive mapping
- Kotlin for Android native implementation
- Objective-C for iOS native implementation
- Mapsforge v0.16.0 for offline map rendering (Android)
- mDNS/Bonjour for automatic device discovery

## Project Structure

### Main Application (`rdzwx-go/`)
- `www/`: Web frontend assets (HTML, JS, CSS, images)
- `www/js/index.js`: Main application logic
- `config.xml`: Cordova configuration
- `platforms/android/`: Generated Android platform code
- `platforms/ios/`: Generated iOS platform code (Xcode project)

### Custom Plugin (`rdzwx-plugin/`)
- `src/android/`: Kotlin native implementation with AIDL interfaces
- `src/android/libs/`: JAR dependencies (Mapsforge, AndroidSVG)
- `src/ios/`: Objective-C native implementation for iOS
- `src/electron/`: Node.js/Electron implementation
- `www/rdzwx.js`: Plugin JavaScript interface
- `plugin.xml`: Plugin configuration

## Development Workflow

1. **Quick Development**: Use `make run` for frontend changes
2. **Plugin Changes**: Use `make full` when modifying native plugin code
3. **Testing**: Deploy to physical Android device or emulator
4. **Prerequisites**: Java 17, Android SDK (API 34+, build-tools 35.0.0), Node.js, Cordova CLI

### iOS Development Workflow
**IMPORTANT**: Plugin changes require reinstalling the plugin, not just building:

```bash
# After making changes to iOS plugin code in rdzwx-plugin/
cd rdzwx-go
cordova plugin remove de-dl9rdz-rdzwx && cordova plugin add plugin-src/rdzwx-plugin/
cordova build ios    # Rebuild iOS app with updated plugin
# Then test in Xcode or simulator
```

**Note**: `cordova build ios` alone does NOT copy plugin changes from `rdzwx-plugin/` to `platforms/ios/`. You must reinstall the plugin first.

### macOS Environment Setup

Required environment variables (add to `~/.zshrc`):

```bash
export JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:/opt/homebrew/bin"
```

Install dependencies:
```bash
brew install openjdk@17 gradle nvm
brew install --cask android-studio
nvm install --lts
npm install -g cordova
```

### Android Build Requirements (cordova-android 14.x)

- **compileSdkVersion**: 35 (required by bundled AndroidX libraries)
- **targetSdkVersion**: 34 (Google Play requirement)
- **Kotlin**: 1.8.22+ (avoids duplicate class conflicts)
- **AIDL**: Enabled via `build-extras.gradle` in plugin (AGP 8.0+ disables by default)

## Release Process

1. Update version in `package.json`, `config.xml`, and `version.json`
2. Run `make release` for signed APK (requires `my-release-key.jks.gpg` keystore)
3. Primary target is Android; Electron for desktop testing

## Key Features

- Real-time radiosonde tracking via network connections
- Offline map support using Mapsforge libraries
- GPS coordinate conversion (WGS84 to EGM)
- Cross-platform tile rendering and caching
- mDNS service discovery for device detection

---

# iOS PORT PROGRESS REPORT

## ✅ COMPLETED (Phase 1: Foundation Setup)

### iOS Platform Integration (December 2024)
- **Complete iOS plugin architecture** implemented in `rdzwx-plugin/src/ios/`
- **Cross-platform compatibility** maintained - both Android and iOS platforms coexist
- **Plugin installation verified** - all source files correctly integrated

### iOS Native Implementation
**Core Classes Implemented:**
- `RdzWx.h/.m` - Main plugin class with all JavaScript interface methods
- `JsonRdzHandler.h/.m` - TCP networking for TTGO ESP32 communication  
- `GPSHandler.h/.m` - Core Location integration with proper permissions
- `MDNSHandler.h/.m` - Bonjour/mDNS service discovery for auto-detection
- `WgsToEgm.h/.m` - Coordinate conversion (WGS84 to EGM96) with bundled geoid data
- `OfflineTileCache.h/.m` - Map tile rendering framework (placeholder implementation)

### Features Ready for Testing
✅ **Network Communication** - TCP socket implementation for JSON-RDZ protocol  
✅ **GPS Integration** - Core Location with permission handling  
✅ **mDNS Discovery** - NSNetServiceBrowser for auto-discovery of TTGO devices  
✅ **Manual Connection** - Direct IP:port connection to TTGO devices  
✅ **Coordinate Conversion** - Geoid height calculation using bundled WW15MGH.DAC  
✅ **Plugin Interface** - All JavaScript methods from Android version ported  

### Configuration Complete
- **iOS Info.plist** configured with required permissions:
  - `NSLocationWhenInUseUsageDescription`
  - `NSLocalNetworkUsageDescription` 
  - `NSBonjourServices` for `_jsonrdz._tcp` discovery
- **Frameworks** linked: CoreLocation, Network, MapKit
- **Resource files** bundled: WW15MGH.DAC geoid data

## 🚧 MAJOR REMAINING WORK

### Phase 4: Offline Mapping (Most Complex - 3-4 weeks)
❌ **Critical Missing Feature** - iOS has no Mapsforge equivalent
- **Challenge**: Android uses Mapsforge Java libraries for .map file rendering
- **iOS Options**:
  1. **Port Mapsforge rendering** to Objective-C/Swift (very complex)
  2. **Pre-generate tile cache** from .map files on desktop (recommended)
  3. **Alternative map format** with conversion tools
- **Current Status**: Placeholder implementation returns empty tiles

### Phase 5: Document Handling (1 week)  
❌ **File Selection** - iOS document picker for .map file selection
- Need to implement `UIDocumentPickerViewController` 
- Handle iOS sandboxing and file access permissions
- Integrate with offline tile cache system

### Phase 6: Performance & Polish (1-2 weeks)
❌ **Background Operation** - iOS restrictions on background networking
❌ **Memory Management** - Optimize for iOS memory constraints  
❌ **App Store Compliance** - Review iOS App Store guidelines

## 🔥 IMMEDIATE NEXT STEPS

### For iOS Device Testing (Required: macOS + Xcode)
1. **Open Xcode Project**: `rdzwx-go/platforms/ios/rdzSonde.xcworkspace`
2. **Build & Deploy** to iOS device
3. **Test Core Features**:
   - GPS location services
   - Manual TTGO connection (IP:port)
   - JSON data parsing and display
   - mDNS auto-discovery
4. **Verify Permissions** - Location and network access prompts

### Expected Test Results
- ✅ **Basic App Launch** - Plugin loads without crashes
- ✅ **GPS Functionality** - Location updates with permission prompts  
- ✅ **Network Communication** - TCP connection to TTGO devices
- ✅ **Data Processing** - JSON parsing and coordinate conversion
- ❌ **Offline Maps** - Will show blank/missing tiles (expected)

## 📋 TESTING CHECKLIST

### Hardware Requirements
- [ ] **TTGO ESP32 device** broadcasting on `_jsonrdz._tcp`
- [ ] **iOS device** (iPhone/iPad) for testing
- [ ] **Same WiFi network** for mDNS discovery
- [ ] **macOS machine** with Xcode installed

### Test Scenarios
- [ ] App launches without crashing
- [ ] Location permission prompt appears and works
- [ ] Network permission prompt appears and works  
- [ ] Manual IP connection to TTGO succeeds
- [ ] Auto mDNS discovery finds TTGO device
- [ ] JSON data from TTGO displays in app
- [ ] GPS coordinates update and display
- [ ] Map view loads (will show blank tiles - expected)

### Known Issues to Expect
- **Offline maps not working** - This is the major remaining work
- **Document picker placeholder** - File selection returns "not implemented"
- **Background limitations** - Connection may drop when app backgrounds

## 🎯 STRATEGIC RECOMMENDATIONS

### Short Term (iOS Device Testing)
1. **Focus on networking** - Verify TTGO communication works
2. **Test GPS integration** - Ensure location services function properly
3. **Validate mDNS discovery** - Auto-detection of devices

### Medium Term (Complete iOS Port)  
1. **Implement offline mapping** - Choose strategy (pre-generation recommended)
2. **Add document picker** - Enable .map file selection
3. **Optimize performance** - Memory usage and background operation

### Long Term (Maintenance)
1. **Maintain dual-platform** compatibility with upstream Android changes
2. **App Store submission** - iOS deployment pipeline
3. **Feature parity** - Ensure iOS matches Android capabilities

---

**Last Updated**: November 2025
**Status**: Both iOS and Android builds working on macOS
**Recent Changes**:
- Android build fixed for cordova-android 14.x (AIDL, Kotlin 1.8.22, compileSdk 35)
- iOS build working via `cordova build ios`
- Environment setup documented in README and CLAUDE.md

---
> Source: [rdzsonde/rdzwx-go](https://github.com/rdzsonde/rdzwx-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
