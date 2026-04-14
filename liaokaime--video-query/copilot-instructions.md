## video-query

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

video-query is a TypeScript library for AI-powered video surveillance analysis. It extracts frames from videos, creates labeled grid mosaics, and uses vision AI models (OpenAI GPT-4 Vision) to locate specific scenes or objects within footage.

## Build Commands

```bash
npm run build      # Compile TypeScript to ESM with type declarations (dist/)
npm run dev        # Watch mode for development
npm test           # Run tests with watch mode (vitest)
npm run test:run   # Run tests once and exit
npm run lint       # Run ESLint
```

## Architecture

**Module Structure:**
```
src/
├── index.ts                    # Barrel exports
├── types/                      # Type definitions (image, video, mosaic, model, query)
└── modules/
    ├── image-input/            # Image data management (ImageInput, ImageItem)
    ├── video-input/            # Video frame extraction (VideoInput, FFmpegExtractor)
    ├── image-mosaic/           # Grid composition (ImageMosaic, LabelRenderer)
    ├── model-adapter/          # AI model integration (BaseModelAdapter, OpenAIAdapter)
    └── query-video/            # Main orchestrator (QueryVideo, ResultParser)
```

**Key Flow:**
1. `QueryVideo` orchestrates all modules
2. Images added via `addImages()` or extracted from video via `addVideo()` using FFmpeg WASM
3. `ImageMosaic` creates labeled grid images with Excel-style coordinates (A1, B2, etc.)
4. `ModelAdapter` sends mosaics to vision AI for analysis
5. `ResultParser` maps AI responses back to original images

**Design Patterns:**
- Adapter Pattern: `BaseModelAdapter` with `OpenAIAdapter` implementation (extensible for other LLM providers)
- Factory Pattern: `createModelAdapter()` function
- Modular Composition: Each module handles single responsibility

## Tech Stack

- TypeScript 5.7 (ES2020 target, strict mode)
- tsup for bundling (ESM output)
- sharp for image processing
- @ffmpeg/ffmpeg (WASM) for video frame extraction
- vitest for testing
- Peer dependency: openai SDK (optional, required for queries)

## Key Types

- `QueryVideo`: Main orchestrator class
- `ImageInput`/`ImageItem`: Image collection management
- `VideoInput`/`FFmpegExtractor`: Video processing
- `ImageMosaic`: Grid composition with position tracking
- `IModelAdapter`/`BaseModelAdapter`: Vision model interface
- `QueryResult`/`MatchedImage`: Query results with confidence scores

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/liaokaime)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/liaokaime)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
