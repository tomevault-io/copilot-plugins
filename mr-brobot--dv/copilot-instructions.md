## dv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**dv** is an interactive document intelligence application built with Dioxus (Rust) that enables users to view documents (PDF, DOCX) and ask AI-powered questions with cited responses.

### Core Vision

- Human-guided document reading with AI assistance (not AI replacement)
- Cross-platform native application (desktop and mobile)
- On-device inference preferred, with remote provider fallback
- Apache-licensed open source project

## Architecture

- **Frontend**: Dioxus 0.6 (cross-platform Rust UI framework)
- **AI**: Qwen-VL for visual intelligence, HuggingFace inference providers
- **Documents**: PDF/DOCX support via URL or local file system
- **Citations**: Content + coordinates (page number, bounding boxes) with navigation

## Development Commands

### Building and Running

```bash
# Check project compiles
cargo check

# Run desktop application
dx serve --platform desktop

# Run web version (requires wasm32 target)
dx serve --platform web

# Build optimized release
cargo build --release
```

### Development Tools

```bash
# Format code
cargo fmt

# Lint code
cargo clippy

# Check for security vulnerabilities
cargo audit
```

## Project Structure

```
├── src/main.rs          # Entry point with basic App component
├── assets/              # Static assets (CSS, icons)
├── Cargo.toml           # Dependencies and platform features
├── Dioxus.toml          # Dioxus-specific configuration
└── .devcontainer/       # Development environment setup
    ├── devcontainer.json
    └── setup.sh         # Automated dependency installation
```

## Platform Features

The project uses cargo features for platform-specific compilation:

- `default = ["desktop"]` - Desktop is the primary target
- `web = ["dioxus/web"]` - Web browser support
- `desktop = ["dioxus/desktop"]` - Native desktop application
- `mobile = ["dioxus/mobile"]` - iOS/Android support (future)

## DevContainer Notes

- Linux dependencies for WebkitGtk are automatically installed via setup script
- Dioxus CLI installed via cargo-binstall for faster setup
- VSCode extensions include Rust-Analyzer and Dioxus support

---
> Source: [mr-brobot/dv](https://github.com/mr-brobot/dv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
