## mindustryreverse

> This project is dedicated to reverse engineering the game **Mindustry** - a tower defense/factory building game. The goal is to analyze, document, and understand the game's mechanics, code structure, and architecture for potential reimplementation.

# Mindustry Reverse Engineering Project - AI Agent Guide

## Project Overview

This project is dedicated to reverse engineering the game **Mindustry** - a tower defense/factory building game. The goal is to analyze, document, and understand the game's mechanics, code structure, and architecture for potential reimplementation.

## Project Goal

**Ultimate Objective**: Create a Rust reimplementation of the Mindustry server that runs seamlessly with existing Java clients (client-server compatibility without client modifications).

**Team Structure**:
- **Analysis Department** (AI + Human): Reverse engineer the Java codebase and produce comprehensive documentation
- **Implementation Department**: Build the Rust server based on analysis documentation

**Analysis Mission**: Create documentation detailed enough that the implementation team can build the Rust server **without needing to read the original Java source code**. Our job is analyzing the original implementation, not interfering with Rust architecture decisions.

**Analysis Approach**:
- **Bottom-up (AI Agent)**: Code-level analysis using MCP tools to extract specifications from Java source
- **Top-down (Human)**: Packet captures, behavioral observation, validation of AI findings
- **Collaborative**: Iterative refinement through feedback between both approaches

## Repository Structure

```
MindustryReverse/
├── Mindustry/          # Git submodule - main Mindustry game repository
├── Arc/                # Git submodule - Arc framework (libGDX fork)
├── analysis/           # Analysis documents and findings
├── build.gradle        # Main build configuration
└── settings.gradle     # Gradle settings with composite builds
```

## Key Components

### Mindustry Submodule
- **Source**: https://github.com/Anuken/Mindustry.git
- **Version**: v152.2-14-g94fc6482e
- **Description**: Main game repository containing all game logic, assets, and modules
- **Key Modules**:
  - `core/` - Core game logic and mechanics
  - `server/` - Dedicated server implementation
  - `desktop/` - Desktop launcher
  - `android/` - Android platform
  - `tools/` - Development tools
  - `tests/` - Test suite

### Arc Submodule
- **Source**: https://github.com/Anuken/Arc.git
- **Version**: v152.2
- **Description**: Game framework - a fork of libGDX with custom modifications
- **Key Modules**:
  - `arc-core/` - Core framework functionality
  - `backends/` - Platform backends (desktop, Android, iOS)
  - `extensions/` - Framework extensions
  - `natives/` - Native library implementations

## Build System

The project uses Gradle with composite builds:

### Available Tasks
- `./gradlew buildAll` - Publishes Arc to Maven Local and builds Mindustry
- `./gradlew buildMindustry` - Compiles only Mindustry core module

### Build Configuration
- Arc and Mindustry are included as composite builds, not standard subprojects
- Arc must be published to Maven Local before building Mindustry (dependency relationship)
- The `analysis/` directory is a standard Gradle subproject (currently contains only documents)

## Development Guidelines

### For AI Agents Working on This Project

1. **Code Analysis Approach**:
   - Use symbolic tools (Serena MCP) for precise Java code navigation within this repository
   - Use semantic search (claude-context MCP) for broader understanding across Mindustry and Arc codebases
   - Mindustry uses many custom patterns - always check existing implementations before suggesting changes

2. **Understanding Dependencies**:
   - Mindustry depends on Arc framework
   - Arc is a modified libGDX - familiar patterns but with custom extensions
   - Check Arc source when unclear about framework behavior

3. **Documentation Strategy**:
   - Place analysis findings in `analysis/` directory organized by phase/system
   - Document specifications and behavior, not implementation details
   - Include code references with file paths and line numbers (format: `file_path:line_number`)
   - Write in language-agnostic manner (pseudocode, formulas, schemas)
   - Focus on "what" and "why", not Java-specific "how"

4. **Reverse Engineering Workflow**:

   **Phase 1: Protocol Layer** (PRIORITY)
   - Document complete network protocol specification
   - Analyze packet serialization in `mindustry.gen.*` (generated code)
   - Map packet handlers and their side effects on server state
   - Document connection lifecycle, handshake, and sync mechanisms
   - *Output*: Protocol specification with packet catalog, structures, serialization rules

   **Phase 2: Core Server Architecture**
   - Trace server startup sequence from `ServerLauncher`
   - Document game loop, tick rate, and update order
   - Identify global state managers and their responsibilities
   - Map threading model and concurrent access patterns
   - *Output*: Architecture document with component diagram, initialization sequence, tick processing

   **Phase 3: Game State & World Model**
   - Document `World`, `Tile`, block grid structures
   - Analyze save/load to identify persistent vs. transient state
   - Map entity system (units, bullets, effects)
   - Distinguish authoritative vs. derived data
   - *Output*: Data model specification with complete state schema and lifecycle rules

   **Phase 4: System-by-System Deep Dives**
   - Priority order: Blocks → Units → Item/Liquid Transport → Power → Combat → Team Management → Waves
   - For each system: architecture, update logic, algorithms, formulas, constants, integration points
   - *Output*: Per-system analysis documents with behavior specifications and edge cases

   **Phase 5: Content System**
   - Document content loading and registration
   - Map base types and inheritance hierarchy
   - Extract stats, properties, and behaviors
   - *Output*: Content specification with type system and property catalogs

## Important Notes

- **Git Submodules**: Both Mindustry and Arc are git submodules. Don't modify them directly in this repo.
- **Version Sync**: Arc and Mindustry versions should remain synchronized (currently both at v152.2)
- **Platform**: Primary development on Windows with IntelliJ IDEA
- **Language**: Mindustry is primarily written in Java
- **Build Time**: Initial builds may take significant time due to asset processing

## Documentation Standards

### Document Templates

Located in `analysis/templates/`:
- **protocol-spec-template.md**: For documenting network packets
- **system-analysis-template.md**: For documenting game systems

### Success Criteria

A document is complete when:
1. ✅ Rust developer can implement the feature without reading Java source
2. ✅ All critical data structures are specified with field types and semantics
3. ✅ All algorithms described in pseudocode, formulas, or state machines
4. ✅ Edge cases and special behaviors documented
5. ✅ Integration points with other systems clearly defined
6. ✅ Constants, magic numbers, and configuration extracted
7. ✅ Validated against packet captures or behavioral tests (where applicable)

### Analysis Directory Structure

```
analysis/
├── README.md              # Analysis overview and navigation
├── templates/             # Documentation templates
│   ├── protocol-spec-template.md
│   └── system-analysis-template.md
├── protocol/              # Network protocol specifications
│   └── README.md
├── architecture/          # Server architecture and core systems
│   └── README.md
├── systems/               # Individual game systems
│   └── README.md
└── content/               # Content system and type definitions
    └── README.md
```

## Useful Commands

```bash
# Update submodules to latest commit
git submodule update --remote

# Check submodule status
git submodule status

# Build everything
./gradlew buildAll

# Run Mindustry desktop (after building)
cd Mindustry && ./gradlew desktop:run
```

## MCP Tools Configuration

This project leverages two MCP servers:

1. **Serena MCP**: For symbolic code navigation and precise editing
2. **claude-context MCP**: For semantic search across large codebases (Mindustry + Arc)

Both codebases should be indexed with claude-context for optimal semantic understanding.

### Generated Code (`mindustry.gen.*`)

Mindustry uses annotation processing to generate code in the `mindustry.gen.*` package:
- **Location**: `Mindustry/core/build/generated/source/kapt/main/mindustry/gen/`
- **Generation**: Run `./gradlew core:compileJava` to generate 262+ Java files
- **Content**: Network packets, entity interfaces, component implementations, etc.

**IMPORTANT**: Generated files are in `build/` directories excluded by .gitignore, so they are **NOT indexed by claude-context MCP**. Use **Serena MCP symbolic tools** to read and analyze generated code. Do not attempt to use semantic search for `mindustry.gen.*` classes.

---
> Source: [lantice3720/MindustryReverse](https://github.com/lantice3720/MindustryReverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
