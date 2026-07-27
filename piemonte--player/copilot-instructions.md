## player

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Player is a simple iOS/tvOS video player library written in Swift. It provides an easy-to-use API for playing local and streaming media with customizable UI and user interaction.

## Build Commands

### CocoaPods
```bash
pod install
```

### Swift Package Manager
```bash
swift build
```

### Xcode Build
```bash
# Build for iOS
xcodebuild -project Project/Player.xcodeproj -scheme "Player" -sdk iphonesimulator build

# Build for tvOS
xcodebuild -project Project/Player.xcodeproj -scheme "Debug - tvOS" -sdk appletvsimulator build

# Run tests (if available)
xcodebuild -project Project/Player.xcodeproj -scheme "Player" -sdk iphonesimulator test
```

### Documentation Generation
```bash
# Generate documentation using jazzy
./build_docs.sh
```

## Architecture

### Core Component
The main library consists of a single Swift file (`Sources/Player.swift`) that implements the `Player` class, which:
- Extends `UIViewController` for easy integration
- Wraps AVFoundation's AVPlayer with a simplified API
- Provides delegate protocols for state management and playback events

### Key Protocols
- **PlayerDelegate**: Handles player state changes (ready, buffering, errors)
- **PlayerPlaybackDelegate**: Manages playback events (time updates, loop events, playback end)

### Primary Features
- Local and HTTP streaming media playback
- Customizable video fill modes (resize, aspect fill, aspect fit)
- Playback state management (stopped, playing, paused, failed)
- Buffering state tracking (unknown, ready, delayed)
- Video frame snapshot support
- Automatic retry on network interruptions
- Picture-in-picture support

### Sample Projects
- `Project/Player/`: iOS sample application
- `Project/PlayerTV/`: tvOS sample application

## Platform Requirements
- iOS 12.0+
- tvOS 12.0+
- Swift 5.0+

## Dependencies
The library has no external dependencies, relying only on iOS/tvOS system frameworks:
- UIKit
- AVFoundation
- CoreGraphics

---
> Source: [piemonte/Player](https://github.com/piemonte/Player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
