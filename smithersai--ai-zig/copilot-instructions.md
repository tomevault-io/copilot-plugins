## ai-zig

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zig AI SDK - A comprehensive AI SDK for Zig, ported from the Vercel AI SDK. Provides unified interfaces for 30+ AI providers including OpenAI, Anthropic, Google, Azure, AWS Bedrock, Mistral, and more. Supports text generation, streaming, tool calling, embeddings, image generation, speech synthesis, and transcription.

**Requirements**: Zig 0.15.0 or later

## Build Commands

```bash
zig build              # Build all packages
zig build test         # Run all unit tests
zig build run-example  # Run the example application
```

There is no way to run individual test files - all tests are compiled and run together via `build.zig`.

## Architecture

### Package Hierarchy

```
packages/
├── provider/           # Core interfaces (LanguageModelV3, EmbeddingModelV3, etc.)
├── provider-utils/     # HTTP client, streaming, memory utilities, SSE parsing
├── ai/                 # High-level API (generateText, streamText, embed, etc.)
└── <provider>/         # Individual provider implementations (openai, anthropic, etc.)
```

### Key Design Patterns

1. **Vtable Pattern**: Interface abstraction for models instead of traits. Each model type (LanguageModelV3, EmbeddingModelV3, etc.) uses vtables for runtime polymorphism.

2. **Callback-based Streaming**: Non-async approach using `StreamCallbacks` with `on_part`, `on_error`, and `on_complete` callbacks.

3. **Arena Allocators**: Request-scoped memory management. Use `defer arena.deinit()` for cleanup.

4. **Provider Pattern**: Each provider implements `init()`, `deinit()`, `getProvider()`, and model factory methods (e.g., `languageModel()`).

### Core Types

- `packages/provider/src/`: `JsonValue` (custom JSON type), error hierarchy, model interfaces
- `packages/provider-utils/src/http/`: HTTP client abstraction and standard library implementation
- `packages/ai/src/generate-text/`: Main text generation with streaming and tool support

## Adding a New Provider

1. Create `packages/<provider-name>/src/` directory
2. Implement provider struct with `init()`, `deinit()`, `getProvider()`, model factory methods
3. Create `index.zig` that re-exports public API with `test { @import("std").testing.refAllDecls(@This()); }`
4. Add module to `build.zig` with appropriate imports (provider, provider-utils, and any base providers)
5. Add test config entry in the `test_configs` array in `build.zig`

## Code Style

- `snake_case` for functions and variables
- `PascalCase` for types
- Doc comments with `///` for public APIs
- Document memory ownership in function signatures
- Use `defer` for cleanup operations

## Memory Management

```zig
var arena = std.heap.ArenaAllocator.init(allocator);
defer arena.deinit();
// Use arena.allocator() for request-scoped allocations
```

Always document whether functions take ownership of allocations or expect the caller to manage memory.

---
> Source: [smithersai/ai-zig](https://github.com/smithersai/ai-zig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
