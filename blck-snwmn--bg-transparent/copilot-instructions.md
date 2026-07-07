## bg-transparent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Bun/TypeScript CLI tool that makes image backgrounds transparent and aligns them to match a reference image's size and position. Uses sharp for image processing.

## Commands

```bash
# Install dependencies
bun install

# Run the CLI
bun run index.ts <input> [options]

# Run tests
bun test

# Run a single test file
bun test src/background-detector.test.ts

# Run tests with coverage
bun test --coverage
```

## Architecture

The codebase follows a modular pipeline architecture for image processing:

```
index.ts (entry point)
    └── src/cli.ts (argument parsing)
    └── src/image-processor.ts (orchestration)
            ├── src/background-detector.ts (color detection)
            ├── src/transparent.ts (transparency application)
            └── src/bounding-box.ts (content bounds detection)
```

### Key Design Patterns

- **Pure data processing**: `processImageData()` handles image transformation without file I/O, making it testable. `processImage()` wraps it with file operations.
- **Raw buffer processing**: Images are processed as raw RGBA buffers for pixel-level manipulation, then converted to PNG via sharp.
- **Reference-based alignment**: Content is scaled and positioned to match a reference image's non-transparent area bounding box.

### Core Types (src/types.ts)

- `RGB`/`RGBA`: Color representations
- `ImageInput`: Raw buffer + metadata for processing
- `ProcessOptions`: Tolerance, color override, reference info
- `BoundingBox`: Rectangle coordinates for content detection

---
> Source: [blck-snwmn/bg-transparent](https://github.com/blck-snwmn/bg-transparent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
