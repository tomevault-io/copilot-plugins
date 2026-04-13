## hello-gen-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow Rules

- When adding a new package or modality, update root README.md, root CLAUDE.md, and create package-level README.md as part of the task.

## Project Overview

Bun monorepo for multi-modal AI experiments. Organized by provider (openai, google) with sub-packages for different modalities (image, transcription, tts, video).

## Commands

```bash
# Install dependencies (from root)
bun install

# Run a specific package
bun run openai/image/index.ts
bun run openai/transcription/index.ts
bun run openai/diarization/index.ts
bun run openai/video/index.ts
bun run google/image/index.ts
bun run google/transcription/index.ts
bun run google/diarization/index.ts
bun run google/tts/index.ts
bun run google/video/index.ts
bun run google/embedding/index.ts <files...>   # embedding → DB保存
bun run google/embedding/search.ts "<query>"   # 類似検索

# Run tests
bun test

# Lint (oxlint with type-aware checking)
bun run lint
bun run lint:fix

# Format (oxfmt)
bun run fmt
bun run fmt:check
```

## Architecture

```
hello-gen-ai/
├── openai/
│   ├── image/            # OpenAI gpt-image-1.5 image editing
│   │   └── index.ts
│   ├── transcription/    # OpenAI Whisper speech-to-text (SRT)
│   │   └── index.ts
│   ├── diarization/      # OpenAI gpt-4o-transcribe-diarize speaker diarization
│   │   └── index.ts
│   └── video/            # OpenAI sora-2 video generation
│       └── index.ts
├── google/
│   ├── image/            # Gemini image generation (pro / flash / Nano Banana 2)
│   │   └── index.ts
│   ├── transcription/    # Gemini gemini-3-flash-preview speech-to-text (SRT)
│   │   └── index.ts
│   ├── diarization/      # Gemini gemini-3-flash-preview speaker diarization
│   │   └── index.ts
│   ├── tts/              # Gemini gemini-2.5-flash-preview-tts text-to-speech
│   │   └── index.ts
│   ├── video/            # Google veo-3.1-generate-preview video generation
│   │   └── index.ts
│   └── embedding/        # Gemini gemini-embedding-2-preview multimodal embedding + similarity search
│       ├── index.ts      # embed files → DB
│       └── search.ts     # similarity search
└── package.json          # Workspace root
```

## Bun Runtime

- Use `bun <file>` instead of `node`
- Use `bun install` instead of npm/yarn/pnpm
- Bun automatically loads `.env` files
- Prefer `Bun.file` over `node:fs` readFile/writeFile
- Use `bun test` with `import { test, expect } from "bun:test"`

## Environment Variables

- `OPENAI_API_KEY` - Required for OpenAI packages (image, transcription, diarization, video)
- `GOOGLE_API_KEY` - Required for Google Gemini packages (image, transcription, diarization, tts, video, embedding)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blck-snwmn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/blck-snwmn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
