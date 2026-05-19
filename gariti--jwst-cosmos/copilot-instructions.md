## jwst-cosmos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JWST Cosmos is a Rust-based TUI (Terminal User Interface) for browsing James Webb Space Telescope images and generating AI-enhanced wallpapers. It connects to remote GPU servers via SSH tunnels for AI processing using ComfyUI and Ollama.

## Build Commands

```bash
# Build debug version
cargo build

# Build optimized release (enables LTO, strips binary)
cargo build --release

# Run directly
cargo run

# Run with debug logging
cargo run -- --debug

# Run with custom config
cargo run -- --config /path/to/config.toml

# Watch for changes and rebuild
cargo watch -x build

# Check for errors without building
cargo check

# Install locally from source
cargo install --path .
```

**Note**: This project currently has no test suite.

## NixOS Integration

The project includes a `flake.nix` for NixOS integration:

```bash
# Enter dev shell with Rust toolchain
nix develop

# Build via Nix
nix build

# Add to NixOS flake inputs
inputs.jwst-cosmos.url = "path:/etc/nixos/jwst-cosmos-rs";
```

## Architecture

```
src/
├── main.rs          # CLI argument parsing, logging setup, app initialization
├── app.rs           # Main TUI event loop, screen management, tunnel coordination
├── config.rs        # TOML configuration with defaults, path expansion utilities
├── screens/         # TUI screens (each implements Screen trait)
│   ├── browser.rs   # Image browsing from ESA/JWST API sources
│   ├── generator.rs # AI image generation form and progress display
│   └── models.rs    # Ollama/ComfyUI model management
├── services/        # Backend service clients
│   ├── jwst_esa.rs  # ESA/Webb RSS feed parser and image downloader
│   ├── jwst_api.rs  # JWST API client (jwstapi.com)
│   ├── ssh_tunnel.rs# SSH tunnel management for remote connections
│   ├── ollama.rs    # Ollama LLM service client
│   ├── comfyui.rs   # ComfyUI WebSocket client for image generation
│   └── wallust.rs   # Wallpaper application and theme refresh
└── utils/
    └── image_utils.rs # Size presets, image manipulation helpers

workflows/           # ComfyUI workflow JSON templates (root level)
├── img2img_sdxl.json
├── controlnet_depth.json
└── controlnet_canny.json
```

### Key Design Patterns

- **Screen Trait**: All TUI screens implement `async_trait Screen` with `draw()` and `handle_key()` methods
- **Service Layer**: Backend services are `Arc`-wrapped for shared ownership across screens
- **SSH Tunneling**: Remote GPU services (Ollama:11434, ComfyUI:8188) accessed via SSH tunnels
- **Async/Await**: Uses `tokio` runtime for async operations, `crossterm` for terminal events

### Data Flow

1. **Browser Screen** fetches images from ESA RSS feed or JWST API
2. Selected images download to `~/Pictures/Wallpapers` (configurable)
3. **Generator Screen** receives reference image path when Tab-switched
4. ComfyUI workflows execute via WebSocket with real-time progress updates
5. **WallustService** applies generated images and triggers theme refresh

## Configuration

Default config location: `~/.config/jwst-cosmos/config.toml`

Key configuration sections:
- `[jwst]` - API key file, wallpaper/cache directories, ESA feed URLs
- `[remote]` - SSH host, user, and port mappings for Ollama/ComfyUI
- `[generation]` - Default size/model, upscaling settings
- `[wallust]` - Auto-apply wallpaper, theme refresh script path

## Workflow Templates

ComfyUI workflows in the root-level `workflows/` directory use placeholder syntax `{{param}}` for dynamic substitution:
- `{{width}}`, `{{height}}` - Output dimensions
- `{{prompt}}` - User text prompt
- `{{model}}` - Checkpoint name
- `{{image}}` - Uploaded reference image filename

## Dependencies Notes

- Uses `rustls-tls` feature for reqwest (no OpenSSL dependency)
- WebSocket via `tokio-tungstenite` for ComfyUI communication
- XML parsing via `quick-xml` for ESA RSS feeds
- `nix` crate for process/signal handling in SSH tunnels

---
> Source: [gariti/jwst-cosmos](https://github.com/gariti/jwst-cosmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
