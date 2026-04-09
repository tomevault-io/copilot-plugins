## codeskew

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build
```bash
cargo build              # Debug build
cargo build --release    # Release build (optimized)
```

### Run
```bash
cargo run -- [OPTIONS] <INPUT_FILE>                # Run with debug build
cargo run --release -- [OPTIONS] <INPUT_FILE>      # Run with release build

# Example: Generate a PNG with 3D perspective
cargo run -- examples/hello.rs -o output.png

# Example: Generate a Telegram sticker (512x512 round PNG)
cargo run -- examples/hello.rs -o sticker.png -f telegram-sticker
```

### Test
```bash
cargo test                     # Run all tests
cargo test test_name          # Run specific test
cargo test -- --nocapture     # Show println! output during tests
```

### Lint & Format
```bash
cargo clippy                  # Run linter
cargo clippy -- -W clippy::all -W clippy::pedantic  # Strict linting
cargo fmt                     # Format code
cargo fmt -- --check         # Check formatting without changes
```

## Architecture

CodeSkew is a WebGPU-based application that renders code snippets with 3D perspective effects. It's designed around a pipeline architecture:

1. **Input Processing**: CLI args → Config struct → File reading
2. **Code Processing**: Syntax highlighting with syntect → Token extraction
3. **Text Layout**: Glyphon integration for glyph positioning and font rendering
4. **3D Rendering**: WebGPU render pipeline with WGSL shaders
5. **Output Generation**: Multiple format encoders (PNG, SVG, GIF, WebP)

### Key Design Patterns

- **Renderer Trait System**: The `webgpu/renderer.rs` defines a trait-based system for different rendering backends
- **Shader Pipeline**: WGSL shaders in `shaders/` are loaded and compiled at runtime, with the toy shader system providing a compute shader framework
- **Async Architecture**: Tokio runtime for non-blocking operations, particularly important for WebGPU initialization
- **Builder Pattern**: Config and renderer setup use builder patterns for flexible configuration

### Module Responsibilities

- `cli.rs`: Command-line parsing with clap, defines all user-facing options
- `config.rs`: Configuration management and validation
- `webgpu/`: Core rendering engine, WebGPU context management
- `glyphon/`: Text rendering integration with cosmic-text
- `output/`: Format-specific encoders, each format has its own module
- `toy/`: Compute shader engine adapted from wgpu-compute-toy project
- `highlight.rs`: Syntax highlighting logic using syntect themes

### WebGPU Specifics

The application requires WebGPU support and uses:
- Vertex/fragment shaders for 3D perspective transformation
- Compute shaders (via toy module) for advanced effects
- Texture sampling for font atlas rendering
- Multiple render passes for compositing

When modifying shaders, ensure WGSL syntax compatibility and test on different backends (Vulkan, Metal, DX12).

## Important Context

1. **Font Assets**: The `fonts/` directory contains FiraCode Nerd Font variants. These are embedded in the binary.

2. **Shader Development**: When modifying shaders in `shaders/`, run with debug build first to catch shader compilation errors early.

3. **Performance**: Release builds are significantly faster due to WebGPU optimizations. Always test performance-critical changes with `--release`.

4. **Output Formats**: Each format has specific requirements:
   - PNG: Standard raster output
   - SVG: Vector format, limited 3D support
   - GIF: Animation support, palette limitations
   - WebP: Modern format with better compression
   - Telegram Sticker: Must be exactly 512x512 pixels, round mask applied

5. **Error Handling**: The codebase uses comprehensive error handling with anyhow. Maintain this pattern for new features.

6. **Zero Warnings Policy**: The project maintains zero compiler warnings. Run `cargo clippy` before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cyrup-ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cyrup-ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
