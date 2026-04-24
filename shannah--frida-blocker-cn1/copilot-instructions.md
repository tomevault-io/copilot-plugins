## frida-blocker-cn1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FridaBlocker CN1 is a Codename One library that provides Frida detection capabilities for mobile applications. It's a **defensive security tool** that helps protect applications from reverse engineering and tampering by detecting the Frida dynamic instrumentation framework.

**IMPORTANT**: This is a defensive security library. Its purpose is to protect legitimate applications from being tampered with or reverse engineered.

## Build System

This project uses Maven with a multi-module structure:

- **Build**: `mvn clean compile` or `mvn clean install`
- **Test**: `mvn test` (runs tests in the `common` module)
- **Package**: `mvn package` (creates the CN1 library)
- **Clean**: `mvn clean`

The project is structured as a parent POM with platform-specific modules:
- `common/` - Shared interfaces and core logic
- `android/` - Android-specific native implementation
- `ios/`, `javascript/`, `javase/`, `win/` - Other platform implementations (currently stub implementations)
- `lib/` - Final CN1 library packaging

## Architecture

### Core Components

1. **FridaBlocker** (`common/src/main/java/ca/weblite/FridaBlocker.java`) - Simple placeholder class
   - Exists mainly for documentation purposes
   - Contains no active functionality

2. **FridaBlockerNativeImpl** (`android/src/main/java/ca/weblite/FridaBlockerNativeImpl.java`) - Android implementation
   - Single static method `detectAndExit()` for Frida detection
   - Called automatically via `android.onCreate` build hint
   - Bridges to native Android FridaBlocker library (`ca.weblite.fridablocker.FridaDetector`)
   - Handles security warnings and app termination on detection

3. **Build Hint Integration** (`common/codenameone_library_appended.properties`)
   - Configures `android.onCreate=ca.weblite.FridaBlockerNativeImpl.detectAndExit()`
   - Automatically injects protection into Android apps

### Platform Support

- **Android**: Full native detection using external FridaBlocker library
- **Other platforms**: No implementation - build hints are ignored

## Key Dependencies

- Codename One framework (v7.0.26)
- External FridaBlocker Android library (ca.weblite.fridablocker.FridaDetector)
- Maven build system

## Development Notes

- Java 8 compatibility required
- Multi-platform Codename One library structure
- Automatic security detection integrated into Android app lifecycle
- Test coverage in `common/src/test/java/ca/weblite/FridaBlockerTest.java`

## Security Context

This library implements multiple Frida detection methods:
- Process scanning
- Port detection  
- Library detection
- File system checks
- Environment variable detection

The Android implementation automatically terminates apps when Frida is detected, showing a security warning dialog.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shannah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
