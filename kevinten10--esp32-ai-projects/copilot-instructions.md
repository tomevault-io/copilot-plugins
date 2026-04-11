## esp32-ai-projects

> This file provides architectural context, development standards, and operational guidelines for the ESP32 AI Projects workspace.

# GEMINI.md - ESP32 AI Projects Context

This file provides architectural context, development standards, and operational guidelines for the ESP32 AI Projects workspace.

## 🚀 Project Overview
This workspace is a collection of AI-assisted ESP32 creative projects using the **PlatformIO** ecosystem and the **Arduino** framework. It is designed for modular development with reusable components and standardized hardware configurations.

- **Primary Hardware**: ESP32 (Puzhong ESP32 / ESP32-WROOM-32), ESP32-S3, ESP32-CAM.
- **Framework**: Arduino (Espressif32 platform).
- **Build System**: PlatformIO.

## 📂 Directory Structure
- **`projects/`**: Individual functional projects (e.g., `weather-station`, `smart-home`). Each project is a standalone PlatformIO project or part of the workspace.
- **`components/`**: Reusable hardware abstraction layers (HAL) and utility modules.
    - `pin-config.h`: **CRITICAL** - Centralized pin mapping for the Puzhong ESP32 board.
    - Modules: `oled-display`, `button-handler`, `sensor-utils`, `wifi-manager`.
- **`docs/`**: Comprehensive setup guides, hardware specs, and AI development prompts.
- **`utils/`**: Helper scripts for environment initialization and maintenance.

## 🛠️ Building and Running
Commands should be executed within the specific project directory (e.g., `projects/weather-station/`).

- **Initialize Project**: `pio init`
- **Compile**: `pio run`
- **Upload**: `pio run --target upload`
- **Monitor Serial**: `pio device monitor` (Default: 115200 baud)
- **Clean Build**: `pio run -t clean`

## 📏 Development Conventions

### 1. Hardware Abstraction & Pin Mapping
- **Always** reference `components/pin-config.h` for GPIO definitions to ensure compatibility with the Puzhong ESP32 board.
- Avoid hardcoding GPIO numbers in `main.cpp`.
- **Restricted Pins**: 
    - GPIO 6-11 (Internal Flash) - **DO NOT USE**.
    - GPIO 34-39 (Input only, no internal pull-up).
    - ADC2 (GPIO 0, 2, 4, 12-15) - **Avoid when WiFi is active**.

### 2. Component Design
- Components should be modular and reside in `components/`.
- Each component should provide a header (`.h`) and implementation (`.cpp`).
- Use constructors to allow overriding default pins defined in `pin-config.h`.

### 3. AI-Assisted Workflow
- Use the prompts in `docs/guides/ai-development.md` for consistent code generation.
- When asking for new features, provide the current `pin-config.h` to the AI to prevent pin conflicts.

### 4. Project Configuration
- Common library dependencies and board settings are managed in the root `platformio.ini`.
- Specific project overrides belong in `projects/<name>/platformio.ini`.

## 🔑 Key Files
- `platformio.ini`: Global build and dependency configuration.
- `components/pin-config.h`: The "Source of Truth" for hardware wiring.
- `projects/weather-station/src/main.cpp`: Reference implementation for a complete project.
- `docs/setup-guide.md`: Detailed environment configuration instructions.

## 🤖 AI Operational Guidance
When assisting with this project:
1. **Verify Pins**: Check `components/pin-config.h` before suggesting any hardware-related code.
2. **Modularize**: Prefer adding functionality to `components/` if it is reusable.
3. **Environment Aware**: Assume a Windows environment with PlatformIO Core installed.
4. **Validation**: Always suggest `pio run` to verify syntax after generating code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinten10)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kevinten10)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
