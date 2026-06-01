## gigi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gigi is a rapid graphics prototyping and code generation suite for real-time rendering techniques. It consists of four main components:

1. **GigiEdit** - Editor for authoring rendering techniques as node graphs (`.gg` files)
2. **GigiViewerDX12** - Viewer for running, debugging, and profiling techniques with Python scripting
3. **GigiCompiler** - Command-line compiler that generates platform-specific code from `.gg` files
4. **GigiBrowser** - Browser for viewing and submitting techniques to the library

The core concept: users describe rendering techniques as graphs of nodes in the editor. Gigi compiles these to human-readable code for various backends (DX12, UE5, WebGPU, Frostbite, etc.).

## Building

Build the entire project:
```bash
# Open and build the solution in Visual Studio
# Both Debug and Release configurations target x64
```

The solution file `gigi.sln` includes all components. After building, executables appear in the root directory.

## Testing

### Viewer Unit Tests
Run from within GigiViewerDX12:
```
File -> Run Python Script -> select Techniques/UnitTests/RunTests.py
```
Should report zero errors.

### DX12 Generated Code Tests
```bash
# Generate code for DX12 unit tests
MakeCode_UnitTests_DX12.bat

# This should produce no diffs. To run the actual tests:
# Open and run _GeneratedCode/UnitTests/DX12/UnitTests.sln
```
Should report zero errors.

### Other Backend Tests
```bash
MakeCode_UnitTests_WebGPU.bat
MakeCode_UnitTests_UE_5_3.bat
MakeCode_UnitTests_FB.bat        # Frostbite
MakeCode_UnitTests_Halcyon.bat
```

## Architecture

### Schema-Driven Design
The project uses `df_serialize` for schema definition. Core types are defined in `Schemas/schemas.h` and processed into C++ types via `Schemas/Types.h`. The schema defines:
- `RenderGraph` - The top-level technique definition
- Node types - ComputeShader, RayShader, DrawCall, CopyResource, SubGraph
- Resource types - Textures, Buffers (imported, internal, exported)
- Variables - Typed values (int, uint, float, float4) with visibility scopes

### GigiCompilerLib
The compiler library (`GigiCompilerLib/`) is the core that processes `.gg` files:
- `gigicompiler.cpp` - Main entry point, calls backend-specific code generation
- `Parse.cpp` - Parses JSON technique files into RenderGraph structures
- `ProcessSlang.cpp` - Handles Slang shader processing
- `structParser.cpp` - Parses C++ structs from shader files
- `SubGraphs.cpp` - Handles subgraph node inlining and loop unrolling

### Backends
Each backend in `GigiCompilerLib/Backends/` generates platform-specific code:
- **DX12/** - DirectX 12 C++ code with full application templates
- **UE_5_3/** - Unreal Engine 5.3 plugin code
- **WebGPU/** - WebGPU code with HTML/JavaScript runners
- **Frostbite/** - EA's Frostbite engine integration
- **Halcyon/** - EA's Halcyon engine integration
- **Interpreter/** - Runtime interpreter backend for the viewer

Backends implement a `PostLoad_<Backend>()` function and generate code via templates in `Backends/<Backend>/templates/`.

### Shader Tokens
Gigi shaders use special tokens for variable access and resource loading:
- `/*$(Variable:VarName)*/` - Access Gigi variables (become constant buffers)
- `/*$(Image2D:file.png:format:type:srgb)*/` - Load textures from disk
- These are processed during compilation and replaced with backend-specific code

### GigiMCP
`GigiMCP/` is a Model Context Protocol (MCP) server that exposes GigiViewerDX12 functionality. It:
- Implements JSON-RPC 2.0 over stdio
- Connects to GigiViewerDX12 via TCP (port 6161)
- Exposes viewer commands as MCP tools (e.g., `mcp__gigi__LoadGG`, `mcp__gigi__RunPythonString`)

### Techniques Directory
- `Techniques/UnitTests/` - Comprehensive unit tests exercising every feature
- `Techniques/DataViewers/` - Viewers for models, textures, PLY files
- Each `.gg` file is a JSON technique definition with a schema reference

## Key Files

- `FlattenRenderGraph.cpp/h` - Flattens subgraphs by inlining and unrolling loops
- `GigiAssert.cpp/h` - Custom assertion handling
- `Schemas/` - All type definitions and schema files
- `UserDocumentation/` - PDFs and docs for shader tokens, Python API, editor usage

## Code Generation Workflow

1. User creates `.gg` file in GigiEdit describing the render graph
2. `.gg` file is JSON validated against `gigischema.json`
3. GigiCompiler parses `.gg` into a `RenderGraph` structure
4. Subgraphs are flattened and optimized
5. Backend-specific `PostLoad_<Backend>()` processes the graph
6. Backend generates code files from templates using graph data
7. Generated code can be compiled and integrated into target applications

## Working with Gigi Files

`.gg` files are JSON with this structure:
```json
{
    "$schema": "gigischema.json",
    "name": "TechniqueName",
    "version": "0.99b",
    "variables": [...],
    "shaders": [...],
    "nodes": [...]
}
```

Nodes reference shaders and resources. The viewer and generated code execute nodes in dependency order.

## External Dependencies

- `external/DirectXTex/` - Texture loading/processing
- `external/slang/` - Slang shader compiler
- `external/AMD/FidelityFX/` - AMD FSR and frame generation
- `external/rapidjson/` - JSON parsing
- `external/imgui-node-editor/` - Node editor UI
- `external/OpenFBX/` - FBX file loading
- `external/df_serialize/` - Schema/serialization framework

## Contributing Notes (from README)

Before submitting pull requests, verify:
1. Viewer unit tests pass (via `Techniques/UnitTests/RunTests.py`)
2. DX12 unit tests pass (via `MakeCode_UnitTests_DX12.bat` producing no diffs)
3. Try to match existing code style
4. Add unit tests when appropriate

---
> Source: [electronicarts/gigi](https://github.com/electronicarts/gigi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
