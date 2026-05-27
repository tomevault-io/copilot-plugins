## hyperguard

> This file serves as the permanent memory for Gemini CLI regarding the Dex2cRemake (HyperGuard) project.

# HyperGuard Project Context

This file serves as the permanent memory for Gemini CLI regarding the Dex2cRemake (HyperGuard) project.

## Project Overview
HyperGuard is a dex2c-based Android protection tool. It converts Dalvik bytecode to C++ code, which is then compiled into native libraries and loaded via a Smali stub.

## Core Architectural Rules (Engineering Standards)

### 1. Build Process
- **Unified Mode**: ALWAYS enabled. Each ABI (armeabi-v7a, arm64-v8a, x86, x86_64) must produce exactly one library file.
- **Library Naming**: Libraries follow the format `lib{base_name}_{hash}_{abi_suffix}.so` where suffix is `a32`, `a64`, `x86`, or `x64`.
- **Force Keep ABIs**: ALWAYS enabled. Do not strip ABIs based on the input APK; always build for all 4 supported architectures.
- **No Virbox Assets**: Do not use the `assets/` folder for library storage as it causes instability. Stick to the standard `lib/` directory.

### 2. Smali Stub Structure (Native.smali)
- **Primary Class**: The main loader class is named `Native` (default: `plongdev.HyperGuardPro.Native`). It has replaced the old `Loader` class.
- **Entry Points**:
    - `attachBaseContext`: Calls `initCore(Context)` native method for early initialization.
    - `clinit`: Calls `loadLibraries()` to dynamically load the correct ABI library.
    - `onCreate`: Initializes `UIHijackingDetect`.
- **String Decryption**: Includes a native method `a([B)Ljava/lang/String;` for runtime string decryption (XOR 0x66).

### 3. UI Hijack Detection (UIHijackingDetect.smali)
- **Features**: Uses `ActivityLifecycleCallbacks` with `WeakReference` to detect when the app is moved to the background or a UI overlay is present.
- **Multi-language**: Supports Vietnamese (default), English, and Chinese.

## File Mapping
- `HyperGuard.py`: Main build script (Modified to support Unified mode and Native stub generation).
- `loader/Native.smali`: The template for the main Smali stub.
- `loader/UIHijackingDetect.smali`: The UI protection logic.
- `project/jni/nc/HyperGuard.cpp`: The native side of the stub (initCore, string decryption, JNI OnLoad).
- `project/jni/Application.mk`: Configured for all 4 ABIs.

## Future Roadmap & Goals (To be implemented)

### 1. Enhanced DEX Protection (HyperShell/dpt-shell Evolution)
- **Zip-Integrity Mimicry**: Improve HyperShell (based on dpt-shell) so that protected DEX files cannot be opened or extracted as ZIP/Archive files (to mimic Virbox's behavior where opening as zip results in an error).
- **In-memory DEX loading**: Refine the transition from ZIP-based storage to raw encrypted data loading.

### 2. Anti-Crack & Integrity
- **Signature Verification**: Add native-level signature checking in `initCore`.
- **Anti-Debug**: Implement `ptrace` checks and other anti-debugging techniques in C++.
- **Environment Checks**: Detect Root, Xposed, and Magisk environments.

### 3. Network & Privacy Security
- **VPN Detection**: Implement logic to detect if a user is running a VPN or Proxy.
- **Developer Options Detection**: Warn or exit if USB Debugging is enabled.

### 4. Technical Context
- **HyperShell**: This component is a renamed and modified version of **dpt-shell**.
- **Goal**: Full encapsulation of the original app logic, leaving only the `Native` stub visible.

---
> Source: [plongdevs/HyperGuard](https://github.com/plongdevs/HyperGuard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
