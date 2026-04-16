## criptogualet

> CriptoGualet is a non-custodial cryptocurrency wallet built with C++. It features a modular architecture with a separation between backend logic and a cross-platform Qt6 GUI.

# CriptoGualet - Gemini Agent Development Guide

## 1. Project Overview

CriptoGualet is a non-custodial cryptocurrency wallet built with C++. It features a modular architecture with a separation between backend logic and a cross-platform Qt6 GUI.

This document provides essential information for an AI agent to understand, build, test, and modify the project.

**Primary Goal:** Assist in developing and maintaining the CriptoGualet application.

## 2. Core Technologies

- **Build System:** CMake (with `CMakePresets.json`)
- **Compiler (Prioritized):** Clang / Clang-CL (on Windows)
- **Compiler (Secondary):** MSVC (on Windows), GCC (on Linux)
- **Dependency Management:** `vcpkg` (see `vcpkg.json` for details)
- **Main Dependencies:**
    - `Qt6`: For the cross-platform GUI.
    - `libqrencode`: For QR code generation.
    - `SQLCipher/SQLite3`: For encrypted database storage.
    - `cpr` & `nlohmann-json`: For blockchain API communication.
    - `secp256k1`: For elliptic curve cryptography.

## 3. How to Build

The project uses `CMakePresets.json`, which defines all necessary build configurations. The recommended preset for development on Windows is `win-vs2022-clangcl-debug`.

### Prerequisites

1.  **Visual Studio 2022:** With the "Desktop development with C++" workload.
2.  **Clang-CL:** Ensure the Clang compiler tools are installed via the Visual Studio Installer.
3.  **vcpkg:** The `VCPKG_ROOT` environment variable must be set.
4.  **CMake:** Ensure CMake is in your system's PATH.
5.  **Qt6:** Install Qt6 (e.g., in `C:/Qt/`). The build scripts will attempt to find it. The `win-vs2022-clangcl-debug` preset is configured to look for a MSVC-compatible build of Qt.

### Build Steps (Command Line)

1.  **Configure:**
    ```bash
    cmake --preset win-vs2022-clangcl-debug
    ```
    This command configures the project in the `out/build/win-vs2022-clangcl-debug` directory.

2.  **Build:**
    ```bash
    cmake --build --preset win-vs2022-clangcl-build
    ```
    This builds the project in `Debug` configuration. The main application executable, `CriptoGualetQt.exe`, will be located in `out/build/win-vs2022-clangcl-debug/bin/`.

## 4. How to Run Tests

Tests are managed by CTest and defined in the `CMakePresets.json`.

### Test Steps (Command Line)

1.  **Ensure the project is built** for debugging (see build steps above).

2.  **Run CTest:**
    ```bash
    ctest --preset win-vs2022-clangcl-test
    ```
    This command will automatically find and run all defined tests for the specified configuration. It provides detailed output on failure.

## 5. Architecture

The project is organized into three main parts: `backend`, `frontend`, and `Tests`.

### Application Entry Point
- The `main()` function is located in `src/CriptoGualetQt.cpp`.
- This file, along with all Qt UI files from `frontend/qt/`, is compiled into the `CriptoGualetQt` executable, as defined in `frontend/qt/CMakeLists.txt`.

### Backend (`backend/`)
- **`core/`**: Handles authentication, session management, and high-level wallet operations.
- **`blockchain/`**: Manages communication with blockchain networks (e.g., BlockCypher API for Bitcoin).
- **`database/`**: Contains the `DatabaseManager` for interacting with the encrypted SQLCipher database.
- **`repository/`**: Provides a data access layer for users, wallets, and transactions.
- **`utils/`**: Contains utilities for QR code generation, secure credential storage, etc.

### Frontend (`frontend/`)
- **`qt/`**: The primary user interface, built with Qt6. Contains all UI components (`Qt*.cpp`/`.h`), and orchestrates the application build.

### Tests (`Tests/`)
- Contains all unit and integration tests. Test executables are defined in `Tests/CMakeLists.txt` and are runnable via CTest.

## 6. Development Notes
- **CMake Debug Path**: For debugging CMake issues, the source directory is the root of the repository.
- **Static Libraries**: All backend components are built as static libraries for security and encapsulation.
- **Automatic DLL Deployment**: On Windows, a post-build command (`windeployqt`) copies necessary Qt and `vcpkg` DLLs to the output directory.
- **Startup Project**: The `CriptoGualetQt` target is the designated startup project in Visual Studio.
- **Compile Commands**: `compile_commands.json` is automatically generated in the build directory for tooling and IntelliSense support.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erickherrera) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
