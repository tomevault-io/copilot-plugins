## squig-model-server

> 1. **NEVER run squig-model-server as a standalone web server** (e.g. `./target/release/squig-model-server`). Always launch it as a **Tauri desktop app** via `./src-tauri/target/release/squig-model-server-desktop`. The environment variable `WEBKIT_DISABLE_DMABUF_RENDERER=1` must be set on NVIDIA + Wayland systems.

# Squig Model Server — Agent Rules

## Mandatory Rules

1. **NEVER run squig-model-server as a standalone web server** (e.g. `./target/release/squig-model-server`). Always launch it as a **Tauri desktop app** via `./src-tauri/target/release/squig-model-server-desktop`. The environment variable `WEBKIT_DISABLE_DMABUF_RENDERER=1` must be set on NVIDIA + Wayland systems.

2. **Build order**: Always build the UI first (`cd ui && npm run build`), then the Rust backend (`cargo build --release`), then the Tauri wrapper (`cd src-tauri && cargo build --release`).

3. **Config file**: Runtime configuration lives in `config.toml` in the project root. Do not hardcode paths or settings — use the config file.

4. **llama.cpp backends**: Per-backend binaries are configured in `[inference.backend_paths]` in `config.toml`. Vulkan build is at `llama.cpp/build/bin/llama-server`, CUDA build is at `llama.cpp/build-cuda/bin/llama-server`.

---
> Source: [hayesjl77/Squig-Model-Server](https://github.com/hayesjl77/Squig-Model-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
