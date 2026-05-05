## machscope

> Auto-generated from feature plans. Last updated: 2026-01-12

# MachScope Development Guidelines

Auto-generated from feature plans. Last updated: 2026-01-12

## Project Overview

MachScope is a native macOS binary analysis tool providing Mach-O parsing, ARM64 disassembly, and optional process debugging. Pure Swift implementation with no external dependencies.

## Active Technologies

- **Language**: Swift 6.2.3 with `SWIFT_STRICT_CONCURRENCY=complete`
- **Frameworks**: Darwin, Foundation, Security (system frameworks only)
- **Platform**: arm64-apple-macosx26.0
- **Build System**: Swift Package Manager

## Project Structure

```text
Package.swift                           # Swift Package manifest (4 targets)
.gitignore                              # Swift/Xcode ignore patterns

Sources/
├── MachOKit/                           # Core Mach-O parsing library
│   ├── MachOBinary.swift               # Main entry point for parsing
│   ├── Header/
│   │   ├── MachHeader.swift            # mach_header_64, FileType, MachHeaderFlags
│   │   ├── FatHeader.swift             # Fat/Universal binary header, FatArch
│   │   └── CPUType.swift               # CPUType, CPUSubtype enums
│   ├── LoadCommands/
│   │   ├── LoadCommand.swift           # Base LoadCommand, LoadCommandType enum
│   │   ├── SegmentCommand.swift        # LC_SEGMENT_64 parsing
│   │   ├── SymtabCommand.swift         # LC_SYMTAB parsing
│   │   ├── DyldCommand.swift           # LC_LOAD_DYLIB and related
│   │   └── CodeSignatureCommand.swift  # LC_CODE_SIGNATURE parsing
│   ├── Sections/
│   │   ├── Segment.swift               # Segment struct, VMProtection
│   │   └── Section.swift               # Section struct, SectionType enum
│   ├── Symbols/
│   │   ├── Symbol.swift                # Symbol struct, SymbolType enum
│   │   ├── SymbolTable.swift           # Lazy symbol table loading
│   │   ├── StringTable.swift           # String table for symbol names
│   │   └── StringExtractor.swift       # Extract strings from sections
│   ├── CodeSignature/
│   │   ├── SuperBlob.swift             # Code signature SuperBlob parser
│   │   ├── CodeDirectory.swift         # CodeDirectory, HashType enum
│   │   └── Entitlements.swift          # XML/DER entitlements parser
│   ├── IO/
│   │   ├── BinaryReader.swift          # Bounds-checked binary reading
│   │   └── MemoryMappedFile.swift      # mmap() wrapper for large files
│   └── Errors/
│       └── MachOParseError.swift       # All parsing error cases with context
│
├── Disassembler/                       # ARM64 instruction decoder
│   ├── ARM64Disassembler.swift         # Main disassembler entry point
│   ├── Instruction.swift               # Instruction model, InstructionCategory
│   ├── Decoder/
│   │   ├── InstructionDecoder.swift    # Base decoder with bit extraction
│   │   ├── DataProcessing.swift        # ADD, SUB, MOV, etc.
│   │   ├── Branch.swift                # B, BL, BR, RET
│   │   ├── LoadStore.swift             # LDR, STR, LDP, STP
│   │   └── System.swift                # SVC, NOP, PAC instructions
│   ├── Formatter/
│   │   ├── InstructionFormatter.swift  # Assembly notation output
│   │   └── OperandFormatter.swift      # Operand display formatting
│   ├── Analysis/
│   │   ├── SymbolResolver.swift        # SymbolResolving protocol
│   │   ├── PACAnnotator.swift          # PAC instruction highlighting
│   │   └── SwiftDemangler.swift        # Swift symbol demangling
│   └── Errors/
│       └── DisassemblyError.swift      # Disassembly error cases
│
├── DebuggerCore/                       # Process debugging (requires entitlements)
│   ├── Debugger.swift                  # Main debugger class
│   ├── Process/
│   │   ├── TaskPort.swift              # task_for_pid wrapper
│   │   ├── ProcessAttachment.swift     # Attach/detach management
│   │   ├── ThreadState.swift           # Thread management
│   │   └── ARM64Registers.swift        # x0-x30, sp, pc, cpsr
│   ├── Breakpoints/
│   │   ├── Breakpoint.swift            # Breakpoint model
│   │   └── BreakpointManager.swift     # Set/remove/hit management
│   ├── Memory/
│   │   ├── MemoryReader.swift          # vm_read wrapper
│   │   └── MemoryWriter.swift          # vm_write wrapper
│   ├── Exceptions/
│   │   ├── ExceptionHandler.swift      # Mach exception handler
│   │   └── MachExceptionServer.swift   # Exception port server
│   ├── Permissions/
│   │   ├── PermissionChecker.swift     # Tiered capability detection
│   │   ├── EntitlementValidator.swift  # Debugger entitlement check
│   │   └── SIPDetector.swift           # SIP status detection
│   └── Errors/
│       └── DebuggerError.swift         # Debugger error cases
│
└── MachScope/                          # CLI executable
    ├── main.swift                      # Entry point, command dispatch
    ├── Commands/
    │   ├── ParseCommand.swift          # machscope parse
    │   ├── DisasmCommand.swift         # machscope disasm
    │   ├── DebugCommand.swift          # machscope debug
    │   └── CheckPermissionsCommand.swift # machscope check-permissions
    ├── Output/
    │   ├── TextFormatter.swift         # Human-readable output
    │   └── JSONFormatter.swift         # Machine-readable JSON output
    └── Utilities/
        └── ArgumentParser.swift        # CLI argument parsing

Tests/
├── MachOKitTests/
│   ├── MachOKitTests.swift             # Core module tests
│   ├── HeaderTests.swift               # MachHeader, CPUType, FileType tests
│   ├── LoadCommandTests.swift          # LoadCommand parsing tests
│   ├── SegmentTests.swift              # Segment and Section tests
│   ├── FatBinaryTests.swift            # Fat/Universal binary tests
│   ├── SymbolTests.swift               # Symbol table tests
│   ├── StringExtractionTests.swift     # String extraction tests
│   ├── ErrorHandlingTests.swift        # Error case tests
│   └── Fixtures/
│       ├── README.md                   # Fixture creation instructions
│       ├── simple_arm64               # Basic ARM64 executable (16KB)
│       ├── fat_binary                 # Universal binary arm64+x86_64 (33KB)
│       ├── signed_binary              # (Phase 8) Code-signed binary
│       └── malformed/
│           ├── truncated              # Truncated header (16 bytes)
│           └── invalid_magic          # Invalid magic number
├── DisassemblerTests/
│   ├── DisassemblerTests.swift         # Core disassembler tests
│   ├── DecoderTests.swift              # ARM64 instruction decoder tests
│   ├── FormatterTests.swift            # Instruction formatter tests
│   ├── PACAnnotatorTests.swift         # PAC annotation tests
│   └── UnknownInstructionTests.swift   # Unknown instruction handling tests
├── DebuggerCoreTests/
│   ├── DebuggerCoreTests.swift         # Core debugger tests
│   ├── PermissionTests.swift           # Permission checker tests
│   └── SIPDetectorTests.swift          # SIP detection tests
└── IntegrationTests/
    ├── IntegrationTests.swift          # Placeholder tests
    ├── ParseIntegrationTests.swift     # Full parsing integration tests
    ├── DisasmIntegrationTests.swift    # Disassembly integration tests
    └── ExportIntegrationTests.swift    # Symbol/string export tests

Resources/
└── MachScope.entitlements              # com.apple.security.cs.debugger, get-task-allow

specs/001-macos-binary-analysis/        # Feature specification
├── spec.md                             # Requirements specification
├── plan.md                             # Implementation plan
├── tasks.md                            # Task breakdown (112 tasks)
├── data-model.md                       # Entity definitions
├── research.md                         # Technical research
├── quickstart.md                       # Development quickstart
├── contracts/
│   └── cli-interface.md                # CLI contract
└── checklists/
    └── requirements.md                 # Spec validation checklist
```

## Commands

```bash
# Build
swift build

# Build release
swift build -c release

# Run tests
swift test

# Run specific test
swift test --filter MachOKitTests

# Run CLI
swift run machscope parse /path/to/binary
swift run machscope parse /path/to/binary --json
swift run machscope disasm /path/to/binary --function _main
swift run machscope check-permissions
swift run machscope debug <pid>

# Code signing for debugging
codesign --force --sign - --entitlements Resources/MachScope.entitlements .build/debug/machscope
```

## Code Style

- Swift 6.2 strict concurrency: All types must be `Sendable`
- No force unwrapping (`!`) in production code
- Domain-specific error types with context (MachOParseError, DisassemblyError, DebuggerError)
- Bounds checking on all buffer access via BinaryReader
- Protocol-oriented design for module boundaries (SymbolResolving, BinaryProviding)
- Memory-mapped file access for binaries >10MB

## Constitution Principles

1. **Security & Permission Handling**: Graceful degradation when permissions unavailable; actionable error messages with System Settings paths
2. **Pure Swift**: No Objective-C bridges unless unavoidable; C interop wrapped in type-safe abstractions
3. **Memory Safety**: Bounds checking on all parsing; no `fatalError()` in production; exhaustive error types
4. **Performance**: mmap() for >10MB files; lazy parsing; memory <2x file size
5. **Modular Architecture**: Protocol boundaries between MachOKit, Disassembler, DebuggerCore; no circular dependencies
6. **Comprehensive Testing**: XCTest with committed fixtures; no system binary dependencies in tests

## Key Files

| Purpose | Location |
|---------|----------|
| Package manifest | `Package.swift` |
| CLI entry point | `Sources/MachScope/main.swift` |
| MachOKit entry | `Sources/MachOKit/MachOBinary.swift` |
| Disassembler entry | `Sources/Disassembler/ARM64Disassembler.swift` |
| Debugger entry | `Sources/DebuggerCore/Debugger.swift` |
| Parse errors | `Sources/MachOKit/Errors/MachOParseError.swift` |
| Disasm errors | `Sources/Disassembler/Errors/DisassemblyError.swift` |
| Debug errors | `Sources/DebuggerCore/Errors/DebuggerError.swift` |
| Entitlements | `Resources/MachScope.entitlements` |
| Test fixtures | `Tests/MachOKitTests/Fixtures/` |
| Constitution | `.specify/memory/constitution.md` |
| Feature spec | `specs/001-macos-binary-analysis/spec.md` |
| Tasks | `specs/001-macos-binary-analysis/tasks.md` |

## Implementation Status

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Setup - Project structure | Complete |
| 2 | Foundational - Core infrastructure | Complete |
| 3 | US1 - Parse Mach-O (MVP) | Complete |
| 4 | US2 - Disassemble ARM64 | Complete |
| 5 | US3 - Check Permissions | Complete |
| 6 | US4 - Debug Process | Complete |
| 7 | US5 - Export Data | Complete |
| 8 | Code Signature Parsing | Complete |
| 9 | Polish & Cross-Cutting | Complete |

**Total Tasks**: 112 | **Parallel Opportunities**: 40

## Module Dependencies

```
MachScope (CLI)
    ├── MachOKit
    ├── Disassembler ──► MachOKit (SymbolResolving protocol)
    └── DebuggerCore ──► MachOKit, Disassembler
```

## File Counts by Module

| Module | Files | Lines (approx) |
|--------|-------|----------------|
| MachOKit | 18 | ~2700 |
| Disassembler | 13 | ~2500 |
| DebuggerCore | 15 | ~4300 |
| MachScope | 8 | ~900 |
| Tests | 19 | ~3100 |
| **Total** | **73** | **~13500** |

## Test Fixtures

Located in `Tests/MachOKitTests/Fixtures/`:

| Fixture | Description | Status |
|---------|-------------|--------|
| `simple_arm64` | Basic ARM64 executable (16KB) | ✓ Created |
| `fat_binary` | Universal binary arm64 + x86_64 (33KB) | ✓ Created |
| `signed_binary` | Code-signed binary with entitlements | Uses simple_arm64 (ad-hoc signed) |
| `malformed/truncated` | Truncated header (16 bytes) | ✓ Created |
| `malformed/invalid_magic` | Invalid magic number | ✓ Created |

## Recent Changes

- 2026-01-12: Phase 9 Polish & Cross-Cutting complete - Enhanced --version flag with build info, --color option (auto/always/never) with NO_COLOR environment variable support, consistent exit codes per cli-interface.md contract, progress indication for large binary parsing, swift-format applied to all sources, end-to-end integration tests, 319 passing tests
- 2026-01-12: Phase 8 Code Signature Parsing complete - SuperBlob parser for code signature container, CodeDirectory parser with CDHash computation, Entitlements parser for XML and DER formats, CodeSignature high-level interface, --signatures and --entitlements flags in ParseCommand, enhanced TextFormatter and JSONFormatter for signature/entitlement output, CodeSignatureTests and EntitlementTests, 319 passing tests
- 2026-01-12: Phase 7 US5 Export Data complete - StringExtractor for extracting strings from __cstring and other sections, ExtractedString model with offset/address tracking, --strings flag in ParseCommand, enhanced TextFormatter and JSONFormatter for string output, export integration tests, 287 passing tests
- 2026-01-12: Phase 6 US4 Debug Process complete - TaskPort wrapper for task_for_pid, ProcessAttachment for attach/detach with ptrace, ThreadState for thread management (read/write registers, single step), ARM64Registers with CPSR flags and Mach thread state conversion, MemoryReader (vm_read) and MemoryWriter (vm_write/vm_protect), Breakpoint model and BreakpointManager, ExceptionHandler and MachExceptionServer for Mach exception ports, Debugger main class integrating all components, DebugCommand with full interactive mode (continue, step, break, info, x, disasm, registers, backtrace), 264 passing tests
- 2026-01-12: Phase 5 US3 Check Permissions complete - SIPDetector for System Integrity Protection status, EntitlementValidator for debugger entitlement and Developer Tools checks, PermissionChecker with tiered capability detection (full/analysis/readOnly), CheckPermissionsCommand with text/JSON output and actionable guidance, 225 passing tests
- 2026-01-12: Phase 4 US2 Disassemble ARM64 complete - Full ARM64 instruction decoder (data processing, branch, load/store, system), PAC instruction annotation, Swift symbol demangling, symbol resolution, InstructionFormatter, CLI disasm command with text/JSON output, 202 passing tests
- 2026-01-12: Phase 3 US1 Parse Mach-O complete - Full MachOBinary parsing, FatHeader support, LoadCommand parsing (17 types), segments/sections, symbol table with lazy loading, CLI parse command with text/JSON output, 96 passing tests
- 2026-01-12: Phase 2 Foundational complete - BinaryReader, MemoryMappedFile, MachOParseError, CPUType, MachHeader, test fixtures
- 2026-01-12: Phase 1 Setup complete - Package.swift, directory structure, entitlements, test fixtures
- 2026-01-12: Analysis passed - ready for implementation
- 2026-01-12: Tasks generated (112 tasks across 9 phases)
- 2026-01-12: Feature specification and planning complete

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Source: [sadopc/machscope](https://github.com/sadopc/machscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
