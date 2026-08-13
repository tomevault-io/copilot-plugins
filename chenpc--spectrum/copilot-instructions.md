## spectrum

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language

Always respond to the user in **Traditional Chinese (繁體中文)**. Technical terms and code identifiers remain in English.

## Build Commands

```bash
./build.sh              # Debug build (incremental)
./build.sh -c           # Clean + Debug build
./build.sh -o           # Build + open app
./test.sh               # Run all tests
./test.sh -t GyroConfigTests                          # Run one test class
./test.sh -t EditOpTests/testRotateThenCrop           # Run single test method
./release.sh            # Release build → Spectrum.dmg
./clean.sh              # Remove build/, DMG, gyro-wrapper/target/
cd gyro-wrapper && cargo build --release              # Rebuild gyroflow-core dylib
```

## Commit Workflow

Always run `./test.sh` before committing. Before committing, consider whether new/modified functionality needs corresponding tests.

## Architecture

**Spectrum** is a native macOS photo/video viewer (SwiftUI + SwiftData + Metal) designed for Sony cameras. It scans existing folders (no import/copy), renders HLG HDR correctly, and provides real-time gyro stabilization via gyroflow-core.

### Data Flow

- **SwiftData models**: `Photo` (media entry) and `ScannedFolder` (bookmarked folder)
- **`FolderScanner`** (`@ModelActor`): accepts `PersistentIdentifier` (not model objects) for cross-actor safety; batch inserts (100/batch) to reduce `@Query` update frequency
- **`Photo.resolveBookmarkData(from:)`**: always use this with `@Query` fallback — `photo.folder?.bookmarkData` can be nil due to SwiftData lazy relationships

### Video Rendering Pipeline

Two-pass Metal pipeline in `AVFMetalView.swift`:
1. **Pass 1**: YCbCr → RGBA16Float (14 decode modes for BT.601/709/2020 × video/full range)
2. **Pass 2**: Warp shader (gyro stabilization with 5 distortion models) → CAMetalLayer drawable

CVDisplayLink dispatches to `renderQueue` (.userInteractive), NOT the real-time thread — prevents CoreAudio priority inversion.

### Gyro Stabilization

- **Single API**: `gyrocore_*` one-shot C API via `GyroCore.swift` (GyroFlowCore was removed)
- **Rust wrapper**: `gyro-wrapper/src/lib.rs` bridges gyroflow-core to Swift via dlopen/dlsym
- **GoPro RS fix**: wrapper detects `detected_source.starts_with("GoPro")` to skip rolling shutter correction (GoPro handles RS internally)
- **matTex format**: width=4 RGBA32Float texture, height=videoH; each row = 3×3 matrix + IBIS + OIS data
- **Config**: `GyroConfig` (Swift) ↔ `Config` (Rust) via snake_case JSON CodingKeys

### HDR

- **Images**: `HDRFormat` enum (.gainMap, .hlg) — detection via `CGImageSourceCopyAuxiliaryDataInfoAtIndex` + `CGColorSpaceUsesITUR_2100TF`
- **Video**: `VideoHDRType` enum — CAMetalLayer colorspace switching (HLG/PQ/sRGB), Dolby Vision P8.4 auto-detect
- **HLG rendering**: CALayer + CGImage directly with `itur_2100_HLG` colorspace (no Core Image)
- **Gain Map**: NSImageView with `preferredImageDynamicRange = .high`

### Key Patterns

- **Security-scoped bookmarks** required in: ThumbnailService, PhotoDetailView, PhotoThumbnailView, FolderScanner, ImagePreloadCache
- **Keyboard navigation**: `@FocusedValue` + menu commands (not `.onKeyPress` — it gets intercepted in NavigationSplitView)
- **`.contentShape()`** needed alongside `.clipped()` to fix hit-testing on `.scaledToFill()` images
- **Non-destructive edits**: `EditOp` enum (crop/rotate/flipH) → `CompositeEdit.from([EditOp])` flattens to final transform; stored in XMP sidecar

### Markdown Style

Never use ASCII art diagrams in markdown documents — use tables, bullet lists, or plain text descriptions instead.

## Development Log

After every bug fix or new feature, append an entry to **`docs/devlog.md`** following this format:

```
## YYYY-MM-DD — <短標題>

**類型：** Bug Fix / Feature / Refactor

**問題：** 一句話描述問題或需求。

**根因／做法：** 說明根本原因或實作方式。

**修改的檔案：** 列出主要修改的檔案。
```

---
> Source: [chenpc/Spectrum](https://github.com/chenpc/Spectrum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
