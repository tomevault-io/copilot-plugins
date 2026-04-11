## xtrustvpn

> **XTrustVPN** is an Android VPN application built upon the **sing-box** universal proxy platform. It leverages a split architecture where the user interface and system integration are handled by Android (Kotlin), while the heavy network lifting, routing, and protocol handling are managed by native Go libraries (`sing-box` wrapped by `libcore`).

# XTrustVPN Project Documentation

## 1. Project Overview

**XTrustVPN** is an Android VPN application built upon the **sing-box** universal proxy platform. It leverages a split architecture where the user interface and system integration are handled by Android (Kotlin), while the heavy network lifting, routing, and protocol handling are managed by native Go libraries (`sing-box` wrapped by `libcore`).

## 2. Directory Structure & Key Files

The project follows a hybrid structure of a standard Android project and a Go workspace.

```text
XTrustVPN/
├── app/                        # Android Application Module
│   ├── src/main/
│   │   ├── java/com/kozak/xtrustvpn/
│   │   │   ├── bg/             # Background Services (VPN Logic)
│   │   │   │   ├── VpnService.kt   # MAIN ENTRY: Android VpnService implementation
│   │   │   │   └── proto/          # Protocol wrappers
│   │   │   │       └── BoxInstance.kt # Kotlin wrapper for Go instance
│   │   │   ├── database/       # Room Database & Preferences (DataStore)
│   │   │   └── ui/             # Activities and Fragments
│   │   ├── assets/             # Pre-packaged assets
│   │   └── res/                # Android Resources (Layouts, Strings)
│   ├── libs/                   # Location where compiled native .aar is placed
│   └── build.gradle.kts        # Android Build Configuration
├── libcore/                    # Go Bridge Library (The "Glue")
│   ├── box.go                  # Core logic to init/start sing-box
│   ├── nb4a.go                 # Initialization for Android environment
│   ├── build.sh                # Script to compile this lib into an AAR
│   └── ...                     # Other Go files for features (DNS, GeoIP)
├── libneko/                    # Go Utility Library
│   ├── protect_server/         # Socket protection (VPN loop prevention)
│   ├── speedtest/              # Latency testing logic
│   └── ...
├── sing-box/                   # The Core Network Engine (Submodule/Source)
├── buildScript/                # Build Automation Scripts
│   ├── init/                   # Environment setup (NDK, Go)
│   └── ...
├── build.gradle.kts            # Root Gradle Project
└── settings.gradle.kts         # Gradle Settings
```

## 3. System Architecture

The application operates in two distinct layers:

### A. The Android Layer (Kotlin)
*   **Responsibility**: UI, Configuration Management, System Lifecycle.
*   **Key Component**: `VpnService` (`app/.../bg/VpnService.kt`).
    *   Extends Android's `VpnService`.
    *   Creates the TUN interface (`Builder.establish()`).
    *   Manages the lifecycle of the Go process.
*   **Data Storage**: Uses `Room` database and `SharedPreferences` (via `DataStore` wrapper) to store profiles and settings.

### B. The Native Layer (Go)
*   **Responsibility**: Traffic routing, Encryption/Decryption, Protocol handling.
*   **Key Component**: `libcore` (Go).
    *   Acts as the JNI bridge using `gomobile`.
    *   **`BoxInstance`** (`libcore/box.go`): The Go struct that holds the running `sing-box` engine.
    *   **`sing-box`**: The actual proxy core that handles `shadowsocks`, `vmess`, `wireguard`, etc.

### C. The Bridge (JNI/Gomobile)
Communication happens via `gomobile` bindings.
*   **Kotlin -> Go**: The Android app calls functions exposed by `libcore`.
    *   Example: `libcore.NewSingBoxInstance(configJson)`
*   **Go -> Kotlin**: The Go code can call back into Kotlin (though mostly used for logging/protect calls).

## 4. Key Logic Flows

### VPN Initialization (Start Sequence)
1.  **User Action**: User taps "Connect" in the UI.
2.  **Service Start**: `VpnService.onStartCommand` is triggered.
3.  **Config Generation**: The app reads settings from the database and generates a JSON configuration string compatible with `sing-box`.
4.  **TUN Creation**: `VpnService` creates the VPN interface (TUN) to intercept network traffic.
5.  **Native Launch**:
    *   Kotlin calls `libcore.NewSingBoxInstance(configJson)`.
    *   `libcore/box.go` parses the JSON.
    *   `libcore` creates a `box.Box` (sing-box instance) with the defined Inbounds (TUN) and Outbounds (Proxy Server).
    *   `instance.Start()` is called.
6.  **Traffic Flow**:
    *   Apps -> Android OS -> TUN Interface -> `sing-box` (Inbound) -> Routing -> Outbound (Proxy) -> Internet.

### VPN Loop Prevention ("Protect")
To prevent the VPN's own traffic from entering the VPN tunnel (infinite loop):
*   `libneko/protect_server` runs a local unix socket server.
*   When `sing-box` opens a socket to connect to a proxy server, it requests protection.
*   The Go layer calls back to Android's `VpnService.protect(socket)` to bypass the VPN.

## 5. Build System

The build is a hybrid process:

1.  **Native Build (`libcore/build.sh`)**:
    *   Sets up the Go environment and NDK.
    *   Uses `gomobile bind` to compile the Go code in `libcore` (and dependencies like `sing-box`) into an Android Archive (`.aar`).
    *   **Output**: `libcore.aar`.
    *   **Destination**: Copies `libcore.aar` to `app/libs/`.

2.  **Android Build (`app/build.gradle.kts`)**:
    *   Standard Gradle build.
    *   Includes `implementation(fileTree("libs"))` to link the generated `libcore.aar`.
    *   Packages everything into the final APK.

## 6. Important Files Analysis

### `app/.../bg/VpnService.kt`
This is the heart of the Android side.
*   `startVpn`: Receives JSON config, builds the `VpnService.Builder`, and initiates the native connection.
*   `killProcesses`: Ensures the file descriptor (`conn`) is closed and native resources released.

### `libcore/box.go`
This is the heart of the Native side.
*   `NewSingBoxInstance`: Factory function. Takes a JSON string, sets up the `sing-box` context with Android-specific registries (loggers, asset managers).
*   `BoxInstance`: Struct wrapping the `sing-box` engine, providing methods like `Start()`, `Close()`, and `UrlTest()` (latency check).

### `libcore/build.sh`
*   Command: `gomobile-matsuri bind ...`
*   Tags: `-tags='with_conntrack,with_gvisor,with_quic,with_wireguard,with_utls,with_clash_api'` - This reveals the enabled features in the core.

## 7. Developer Notes
*   **Modifying Core Logic**: Changes to routing or protocols must be done in Go (`libcore` or `sing-box`), then `libcore` must be recompiled (`libcore/build.sh`) before the Android app can see them.
*   **Modifying UI**: Standard Android development in `app/src/main`.
*   **Debugging**: Logs from the Go side are piped to Android logcat via `boxPlatformLogWriter` in `libcore`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kozak809)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kozak809)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
