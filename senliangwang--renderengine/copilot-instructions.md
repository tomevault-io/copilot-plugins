## renderengine

> C++17 OpenGL game engine (Hazel Engine) with a CAD-style 3D cross-section Sandbox demo. Single native desktop app — no web services, databases, or containers.

# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

C++17 OpenGL game engine (Hazel Engine) with a CAD-style 3D cross-section Sandbox demo. Single native desktop app — no web services, databases, or containers.

### Build

```bash
cd /workspace
mkdir -p build && cd build
cmake -DCMAKE_CXX_COMPILER=g++ -DCMAKE_C_COMPILER=gcc -DGLFW_BUILD_WAYLAND=OFF ..
make -j$(nproc)
```

- `-DGLFW_BUILD_WAYLAND=OFF` is required because wayland-scanner is not available in the Cloud VM.
- The binary is produced at `build/bin/Sandbox`.

### Running the Application

The Sandbox requires an X11 display with OpenGL support. In the Cloud VM, use Xvfb with software rendering:

```bash
Xvfb :99 -screen 0 1280x1024x24 -ac +extension GLX +render -noreset &
export DISPLAY=:99
cd /workspace/Sandbox
LIBGL_ALWAYS_SOFTWARE=1 ../build/bin/Sandbox
```

- `LIBGL_ALWAYS_SOFTWARE=1` forces Mesa llvmpipe software rendering (no GPU in Cloud VM).
- The working directory must be `/workspace/Sandbox` so the app can find `assets/shaders/`.

### Lint / Tests

This project has no dedicated linter or test suite. A successful `cmake` configure + `make` build with zero errors is the primary correctness check.

### Key Gotchas

- Git submodules (`spdlog`, `GLFW`, `GLM`) must be initialized before building: `git submodule update --init --recursive`.
- System packages `libxrandr-dev`, `libxinerama-dev`, `libxcursor-dev`, `libxi-dev`, `libgl1-mesa-dev` are needed for GLFW/OpenGL compilation.
- Chinese characters in ImGui may render as `????` if the font doesn't include CJK glyphs — this is cosmetic, not a bug.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SenliangWang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SenliangWang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
