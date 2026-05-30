## phoneproxy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Install on connected device
./gradlew installDebug

# Run unit tests
./gradlew test

# Run instrumented tests (requires device/emulator)
./gradlew connectedAndroidTest

# Run a single unit test class
./gradlew test --tests "com.fokinvd.phoneProxy.ExampleUnitTest"

# Lint check
./gradlew lint
```

## Project Overview

**PhoneProxy** — Android app (minSdk 24, targetSdk 36) that runs an HTTP proxy server on the device, allowing hotspot-connected devices to route their traffic through the host device's active VPN tunnel. The user manages VPN separately; this app only manages the proxy layer.

## Architecture

Single-module Kotlin + Jetpack Compose app. Package: `com.fokinvd.phoneProxy`.

**Key layers:**
- `ui/` — Compose screens and ViewModels (MVVM pattern)
- `ui/theme/` — Material3 theme, colors, typography
- `proxy/` — `ProxyServer` (pure Kotlin TCP/HTTP proxy logic)
- `service/` — `ProxyService` (foreground service wrapping `ProxyServer`)
- `data/` — `PreferencesRepository` (DataStore-backed settings)

**Data flow:** UI → ViewModel → starts/stops `ProxyService` → `ProxyService` owns `ProxyServer` lifecycle. Settings are persisted via DataStore and read by both the ViewModel (for display) and `ProxyServer` (for port binding).

## Key Design Decisions

- **HTTP CONNECT proxy** (not SOCKS5): natively supported in Android/iOS hotspot proxy settings without extra client apps.
- **Foreground service**: required to keep the proxy alive when the screen is off; posts a persistent notification with current proxy address.
- **Hotspot IP detection**: reads the `wlan0` interface address to show the correct IP to configure on client devices.
- **No root required**: standard socket binding on a user-space port (≥1024).

## Dependencies

Managed via `gradle/libs.versions.toml`. Key additions beyond the starter template:
- `androidx.datastore:datastore-preferences` — settings persistence
- `androidx.lifecycle:lifecycle-service` — `LifecycleService` for the foreground service
- `androidx.compose.runtime:runtime-livedata` or `lifecycle-runtime-compose` — state collection in Compose

---
> Source: [mareliberum/PhoneProxy](https://github.com/mareliberum/PhoneProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
