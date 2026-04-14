## pi-agent-global

> This is the **rb-openscreen-zig-dsl** project — a Ruby DSL that serves as the architectural specification and single source of truth for the [openscreen-zig](https://github.com/nickptar/openscreen-zig) project (a Zig rewrite of OpenScreen). The DSL lives in `openscreen_zig_dsl.rb` at the project root and defines every component, dependency, allocator strategy, and platform target for the Zig codebase.

# OpenScreen-Zig DSL — Project Instructions

## What This Project Is

This is the **rb-openscreen-zig-dsl** project — a Ruby DSL that serves as the architectural specification and single source of truth for the [openscreen-zig](https://github.com/nickptar/openscreen-zig) project (a Zig rewrite of OpenScreen). The DSL lives in `openscreen_zig_dsl.rb` at the project root and defines every component, dependency, allocator strategy, and platform target for the Zig codebase.

## Project Structure

```
rb-openscreen-zig-dsl/
├── openscreen_zig_dsl.rb     # The Ruby DSL — single source of truth for architecture
├── AGENTS.md                 # This file — project instructions for Pi
├── .pi/
│   ├── settings.json         # Pi project-level settings
│   ├── SYSTEM.md             # Pi system prompt
│   ├── skills/               # Pi skills (dsl-query, zig-generate, dependency-trace, platform-target)
│   ├── prompts/              # Prompt templates (review, architecture, impact)
│   ├── convex/
│   │   └── schema.ts         # Convex database schema (reference/documentation)
│   └── README.md             # Pi agent documentation (reference)
└── src/                      # Generated Zig source output directory
```

## The DSL File: `openscreen_zig_dsl.rb`

This is the most important file in the project. It defines:

- **Domains** — top-level architectural areas (CapturePipeline, AudioPipeline, RecordingEngine, ZoomEngine, TimelineEditor, AnnotationSystem, CompositingEngine, ExportPipeline, CursorTelemetry, ProjectPersistence, PlatformLayer)
- **Components** — individual modules within domains
- **Relationships** — how components connect via the DSL vocabulary

### DSL Vocabulary

| Keyword | Meaning |
|---|---|
| `depends_on` | What a component needs to function |
| `edited_by` | What mutation system modifies it |
| `touches` | What else changes when this changes |
| `renders_with` | Which rendering components display this |
| `ui_behavior` | UI interaction patterns (overflow, content count, empty states) |
| `allocates_with` | Zig allocator strategy (gpa, arena, page_allocator, etc.) |
| `comptime_generates` | What is produced at comptime |
| `cross_compiles_to` | Target platforms (macos, windows, linux) |

### Component Properties

Each component also tracks:
- `zig_imports` — `@import` paths
- `error_set` — explicit Zig error types
- `platform_specific` — whether the component has platform branches
- `file_path` — corresponding Zig source file
- `public_api` — exported functions/types
- `internal_state` — struct fields

## Conventions

### Zig Code Conventions

When generating or reviewing Zig code for this project:

1. **Explicit allocators** — Every function that allocates takes an `std.mem.Allocator` parameter. No hidden allocations.
2. **Error unions** — Use `!ReturnType` or explicit error sets. All errors must be propagated with `try` or caught with `catch`.
3. **Comptime** — Use `comptime` for platform switches, type generation, and build-time configuration.
4. **Naming** — Component names in the DSL map to Zig file names and struct names (e.g., `ScreenCapture` → `screen_capture.zig` → `ScreenCapture` struct).
5. **Platform guards** — Use `@hasDecl` or build.zig options for platform-specific code paths.
6. **C interop** — Use `@cImport` for system library bindings (CoreGraphics, DXGI, X11, etc.).
7. **Formatting** — Always run `zig fmt` on generated code.

### DSL Conventions

- The DSL is the source of truth. If architecture changes, update `openscreen_zig_dsl.rb` first.
- Component names are PascalCase and unique across all domains.
- Domain names are PascalCase.
- File paths in the DSL are relative to the Zig project root (e.g., `src/capture/screen_capture.zig`).
- Platform names are lowercase atoms: `macos`, `windows`, `linux`.

### Allocator Strategy Mapping

| DSL Value | Zig Type | Use Case |
|---|---|---|
| `GeneralPurposeAllocator` | `std.heap.GeneralPurposeAllocator` | General use, leak detection |
| `ArenaAllocator` | `std.heap.ArenaAllocator` | Batch operations, frame processing |
| `PageAllocator` | `std.heap.page_allocator` | Large allocations, system-level |
| `FixedBufferAllocator` | `std.heap.FixedBufferAllocator` | Embedded, no-heap scenarios |

## Build Commands

```bash
# Syntax-check the DSL
ruby -c openscreen_zig_dsl.rb

# Run the DSL to dump its JSON representation
ruby openscreen_zig_dsl.rb

# Format Zig code (run on any generated .zig files)
zig fmt src/

# Build the Zig project (from the openscreen-zig repo)
zig build

# Run Zig tests
zig build test

# Build for specific target
zig build -Dtarget=x86_64-windows-gnu
zig build -Dtarget=aarch64-linux-musl

# Cross-compile for all platforms
zig build -Dtarget=x86_64-macos   # macOS Intel
zig build -Dtarget=aarch64-macos  # macOS Apple Silicon
zig build -Dtarget=x86_64-windows-gnu  # Windows
zig build -Dtarget=x86_64-linux-musl   # Linux
```

## Convex Integration

The project includes a Convex schema at `.pi/convex/schema.ts` for reference. It defines tables for:
- `components` — registry of all DSL-defined components
- `dependencies` — directed edges in the dependency graph
- `agentSessions` — session tracking
- `changeHistory` — audit trail
- `dslSnapshots` — versioned DSL state

Convex is optional — the DSL file itself is the primary knowledge store.

## Common Operations

### Query the Architecture
Read `openscreen_zig_dsl.rb` and trace the relevant `depends_on`, `touches`, or `renders_with` relationships.

### Add a New Component
1. Add a `component` block inside the appropriate `domain` in `openscreen_zig_dsl.rb`
2. Set `depends_on`, `allocates_with`, `cross_compiles_to`, and other properties
3. Set `file_path` to where the Zig source will live
4. Generate the Zig skeleton in `src/`

### Modify an Existing Component
1. Update the component's properties in the DSL
2. Check `touches` and `edited_by` to find affected components
3. Update those components' Zig source files
4. Run tests

### Platform-Specific Implementation
1. Check the component's `cross_compiles_to` property
2. Generate platform-specific code using comptime switches or separate files
3. Use `@cImport` for platform APIs:
   - **macOS**: CoreGraphics, CoreAudio, AVFoundation, ScreenCaptureKit
   - **Windows**: DXGI, WASAPI, Graphics Capture API, DirectShow
   - **Linux**: X11/RANDR, Wayland, PipeWire, PulseAudio, ALSA

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stussysenik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
