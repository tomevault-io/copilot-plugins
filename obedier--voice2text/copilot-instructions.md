## voice2text

> This file documents the development environment setup and build process for the VoiceDictate application.

# VoiceDictate Development Environment Rules

This file documents the development environment setup and build process for the VoiceDictate application.
Use this information when assisting with development of this project.

## Development Environment

1. Swift Version:
   - Use Swift 5.9 or later
   - Install via Homebrew: `brew install swift`
   - Add to PATH: `export PATH="/opt/homebrew/opt/swift/bin:$PATH"`

2. macOS Requirements:
   - Target macOS 13.0 or later
   - Developer must have Command Line Tools installed: `xcode-select --install`

3. Dependencies:
   - Defaults package: https://github.com/sindresorhus/Defaults (for user preferences management)

## Project Structure

1. Source Organization:
   - All source code in `src/VoiceDictate/Sources/`
   - Test files in `src/VoiceDictate/Tests/`
   - Info.plist must be in `src/VoiceDictate/Sources/Info.plist`

2. Entry Point:
   - Traditional AppKit entry point in main.swift
   - No SwiftUI App @main implementation (causes build conflicts)

3. Build Artifacts:
   - Debug builds in `.build/debug/`
   - Release builds in `.build/release/`
   - App bundle in `build/VoiceDictate.app/`

## Build Process

1. Command Line Build:
   - Debug build: `swift build`
   - Release build: `swift build -c release`
   - Full app bundle creation: `./build_app.sh`

2. Key Build Steps:
   - Compile Swift sources using Swift Package Manager
   - Create macOS app bundle structure (Contents/MacOS, Contents/Resources)
   - Copy executable to Contents/MacOS
   - Copy Info.plist to Contents/
   - Create app resources in Contents/Resources

3. Common Build Issues:
   - Swift version mismatches between SDK and compiler
   - Multiple @main declarations causing conflict
   - AVAudioSession APIs not available in macOS (use AVAudioEngine directly)
   - Info.plist placement (must be in Sources/ and excluded in Package.swift)

## Package.swift Configuration

1. Critical Settings:
   - Swift tools version 5.9
   - Target macOS 13.0 or later
   - Executable target with proper path
   - Resources declaration
   - Info.plist must be excluded

2. Example:
```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "VoiceDictate",
    platforms: [.macOS(.v13)],
    products: [.executable(name: "VoiceDictate", targets: ["VoiceDictate"])],
    dependencies: [.package(url: "https://github.com/sindresorhus/Defaults", from: "7.1.0")],
    targets: [
        .executableTarget(
            name: "VoiceDictate",
            dependencies: ["Defaults"],
            path: "src/VoiceDictate/Sources",
            exclude: ["Info.plist"],
            resources: [.process("Resources")]
        ),
        .testTarget(
            name: "VoiceDictateTests",
            dependencies: ["VoiceDictate"],
            path: "src/VoiceDictate/Tests"
        )
    ]
)
```

## macOS App Bundle Creation

1. App Bundle Structure:
```
VoiceDictate.app/
├── Contents/
│   ├── Info.plist
│   ├── MacOS/
│   │   └── VoiceDictate (executable)
│   └── Resources/
│       └── [resource files]
```

2. Required Info.plist Keys:
   - CFBundleExecutable: "VoiceDictate"
   - CFBundleIdentifier: "com.example.VoiceDictate"
   - CFBundlePackageType: "APPL"
   - NSPrincipalClass: "NSApplication"
   - LSUIElement: true (for menu bar app)
   - NSMicrophoneUsageDescription and NSSpeechRecognitionUsageDescription for permissions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/obedier) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
