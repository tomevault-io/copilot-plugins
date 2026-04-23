## transform-images

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Quak Images** is a desktop application for batch image optimization, conversion, and transformation. It's built with:
- **Frontend**: SvelteKit 2 + TypeScript + Tailwind CSS 4 + Vite (SPA mode with adapter-static)
- **Backend**: Tauri 2 + Rust with Clean/Hexagonal Architecture
- **Image Processing**: libraw (RAW formats), mozjpeg, oxipng, libwebp

The app handles RAW formats from Sony/Canon/Nikon and provides advanced transformations (resize, rotate, flip) with multithreaded processing and cancellation support.

## Development Commands

### Frontend Only
```bash
bun run dev          # Start Vite dev server on port 1420
bun install          # Install dependencies (or npm install)
bun run build        # Build frontend to dist/ with TypeScript checking
```

### Full App (Frontend + Rust Backend)
```bash
bun run tauri:dev    # Development mode with hot reload
bun run tauri:build  # Production build for current OS
```

### Backend Testing
```bash
cd src-tauri
cargo test           # Run all tests
cargo test --lib     # Run library tests only (domain + infrastructure)
```

### Cross-Platform Build
```bash
# Add target first
rustup target add aarch64-apple-darwin

# Build for specific target
bun run tauri build --target aarch64-apple-darwin --features static
```

## Architecture

### Backend (Rust) - Clean/Hexagonal Architecture

The Rust backend follows Domain-Driven Design with clear separation:

```
src-tauri/src/
├── domain/              # Business logic (framework-agnostic)
│   ├── models/          # Image, ProcessingSettings, Transformation
│   ├── value_objects/   # ImageFormat, Quality, Dimensions
│   ├── services/        # ImageProcessor trait
│   └── error.rs         # DomainError
│
├── application/         # Use cases & Tauri commands
│   ├── commands.rs      # Tauri command handlers exposed to frontend
│   ├── state.rs         # AppState with Arc<Mutex<TaskManager>>
│   ├── task_manager.rs  # Cancellation & progress tracking
│   └── dto.rs           # Data transfer objects
│
└── infrastructure/      # External implementations
    ├── image_processor/
    │   ├── processor_impl.rs     # ImageProcessorImpl (main processor)
    │   ├── raw_processor.rs      # RawProcessor (libraw FFI)
    │   ├── batch_processor.rs    # Parallel batch processing with rayon
    │   ├── optimizers/           # Format-specific optimizers
    │   │   ├── png_optimizer.rs  # oxipng
    │   │   ├── jpeg_optimizer.rs # mozjpeg
    │   │   └── webp_optimizer.rs # libwebp
    │   └── transformers/
    │       ├── resizer.rs        # Lanczos, Triangle, etc.
    │       └── rotator.rs        # Rotate & flip
    ├── metadata_cleaner.rs       # EXIF removal
    └── file_system/              # File operations
```

**Processing Flow**:
1. Frontend calls Tauri command (e.g., `process_images`) → `application/commands.rs`
2. Command validates inputs and creates `ProcessingSettings` (domain model)
3. `ImageProcessorImpl` loads image:
   - If RAW format → `RawProcessor` decodes with libraw
   - Otherwise → standard image crate
4. Apply transformations (`transformers/resizer.rs`, `transformers/rotator.rs`)
5. Apply format-specific optimizer (`optimizers/`)
6. `MetadataCleaner` removes EXIF before writing final file

### Frontend (TypeScript/Svelte)

```
src/
├── lib/
│   ├── components/
│   │   ├── header/          # Logo, ActionButtons
│   │   ├── sidebar/         # FormatSelector, QualitySlider, TransformationsPanel
│   │   └── main/            # DropZone, ImageGrid, ProgressBar, ResultsPanel
│   ├── app/services/        # ImageService (Tauri command wrappers)
│   ├── app/state/           # AppState (reactive state management)
│   └── models/types.ts      # TypeScript type definitions
└── routes/                  # SvelteKit routes
```

**Key Frontend Pattern**:
- `ImageService` wraps all Tauri commands (`invoke()`)
- `AppState` manages reactive state with Svelte 5 runes
- `DropZone` handles drag-and-drop for files/folders
- Progress tracking via polling `get_processing_status` command

## Important Patterns

### Tauri Commands
All Tauri commands are defined in `src-tauri/src/application/commands.rs` and registered in `lib.rs`. Commands are async and use `State<AppState>` for shared state.

### Cancellation
The `TaskManager` uses an `AtomicBool` for cancellation. Batch processing checks `should_cancel()` periodically via `batch_processor.rs`.

### Static Linking
The `static` feature flag enables static linking of libraw for distribution on macOS/Windows (see `Cargo.toml` features). Linux uses dynamic linking against system libraries.

### Frontend Build for Tauri
Tauri expects the frontend build in `dist/` with `index.html` as the entry point. The SvelteKit adapter is configured for SPA mode (single-page app) since Tauri doesn't support SSR.

Set `TAURI_SKIP_BUILD_FRONTEND=true` in CI when the frontend is pre-built to avoid rebuilding.

## Version Management

**CRITICAL**: Version is managed in **TWO** locations and must be kept in sync:
- `package.json` → `"version": "0.6.2"`
- `src-tauri/Cargo.toml` → `version = "0.6.2"`
- `src-tauri/tauri.conf.json` → `"version": "0.6.2"`

When bumping version, update all three files simultaneously.

## CI/CD Pipeline

GitHub Actions workflow (`.github/workflows/build-release.yml`):
1. **Test**: Run `cargo test --lib` on Ubuntu
2. **Build Frontend**: Build Svelte app once, cache by content hash, upload as artifact
3. **Build Tauri**: Build for 5 platforms in parallel (macOS Intel/ARM, Windows x64, Linux x64/ARM64)
4. **Release**: On `v*` tags, create GitHub release with renamed binaries and `latest.json` for auto-updates

Platform-specific build args:
- macOS/Windows: `--features static` (static libraw linking)
- Linux: No features (uses system libraries)

## Testing

- Unit tests: `cargo test --lib` (domain + infrastructure)
- Integration tests: `src-tauri/tests/` (RAW conversion, transformations)
- Manual testing: Drag & drop, cancellation, progress updates, metadata preservation

## Dependencies

### Rust
- `libraw-sys`: Custom FFI bindings in `src-tauri/vendor/libraw-sys`
- Image optimizers: `oxipng`, `mozjpeg`, `webp`
- Async: `tokio` (full features)
- Parallelism: `rayon` for batch processing

### Frontend
- `@tauri-apps/api`: Tauri JS bindings
- `@tauri-apps/plugin-*`: Dialog, opener, process, updater plugins
- `@tailwindcss/vite`: Tailwind CSS 4 (Vite plugin)

## Common Issues

### "Asset not found: index.html"
Ensure `dist/` exists with `index.html` before running `tauri build`. Run `bun run build` first, or set `TAURI_SKIP_BUILD_FRONTEND=true` if using pre-built frontend.

### macOS Gatekeeper Blocking App
Users without developer certificates need to run:
```bash
xattr -cr "/Applications/Quak Images.app"
```

### LibRaw Build Failures
On macOS: `brew install libraw little-cms2 jpeg-turbo`
On Linux: `sudo apt-get install libraw-dev liblcms2-dev libjpeg-dev`

## Code Style

- Rust: Follow standard Rust conventions, prefer `anyhow::Result` in infrastructure, `DomainResult` in domain
- TypeScript: Strict mode enabled, no implicit `any`
- Keep domain layer pure (no Tauri dependencies)
- Use `thiserror` for domain/infrastructure errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DereckAn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
