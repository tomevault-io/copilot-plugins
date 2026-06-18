## nexu

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Nexu** is a cross-platform **Avalonia (.NET 10) desktop application** — a "Zen-mode mind-map" editor for JSON files.

## Stack

- **Platform:** .NET 10, Avalonia UI
- **IDE:** JetBrains Rider
- **Package manager:** NuGet

## Solution Structure

```
Nexu.sln
├── src/
│   ├── App                  # Avalonia entry point, canvas host, pan/zoom, selection
│   ├── Domain               # NodeGraph model (NodeId, NodeType, Parent/Children)
│   ├── Parsing.Json         # JSON CST parser — nodes with start/end char indices
│   ├── Persistence          # Atomic file writer, autosave, save blocking
│   └── Layout               # Deterministic left-to-right layout engine
└── tests/
    ├── Tests.Domain
    ├── Tests.Parsing.Json
    ├── Tests.Persistence
    └── Tests.Layout
```

## Build & Test Commands

```bash
dotnet build                                          # build solution
dotnet test                                           # run all tests
dotnet test --filter "FullyQualifiedName~MyTest"      # run a single test
dotnet run --project src/App                          # run the desktop app
```

## Architecture & Core Design Principles

**Raw text is the source of truth.** Edits are never done by deserializing and reserializing JSON. Instead:

1. JSON is parsed into a **CST** (Concrete Syntax Tree) — every node carries `Start` and `End` character indices into the original string.
2. The CST is mapped to a **NodeGraph** (`Domain` layer) — `NodeId`, `NodeType`, parent/children relationships.
3. The NodeGraph is passed through a **deterministic layout engine** (`Layout`) producing `PositionedNode` + `Edge` values.
4. The Avalonia **canvas** (`App`) renders positioned nodes with pan/zoom and selection.
5. User edits are expressed as typed **EditIntents**: `RenameProperty`, `SetScalarValue`, `AddProperty`, `AddArrayItem`, `RemoveNode`.
6. Each intent is converted by the **TextPatch generator** into `{ Start, Length, OldText, NewText }` — a surgical character-level replacement applied to the raw string.
7. Patches are stored on a **history stack** for undo/redo.
8. **Persistence** writes atomically (write to temp file, then replace).

**Save is blocked** if the CST detects duplicate keys or unresolved syntax errors.

## Development Model

- Trunk-based development — one issue → one branch → one PR → squash merge
- TDD-first — write tests before implementation
- Small, isolated issues (≤3 files per PR ideally)
- CI must be green before merge (build + test + no warnings — warnings are errors)
- Nullable enabled, `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`

## Git Rules

- Commit often, small atomic changes. Format: `[type] what and why`
- `commit` = do it yourself (Tier 1). `push` = requires explicit user confirmation (Tier 3).

---

# === COGNILAYER (auto-generated, do not delete) ===

## Memory Tools
You have access to the `cognilayer` MCP server:
- memory_search(query) — search memory semantically
- memory_write(content) — save important information
- file_search(query) — search project files (PRD, docs...)
- decision_log(query) — find past decisions

When unsure about context or project history,
ALWAYS search memory first via memory_search.
When you need info from PRD or docs, use file_search
INSTEAD of reading the entire file.

## VERIFY-BEFORE-ACT — MANDATORY
When memory_search returns a fact marked with ⚠ STALE:
1. ALWAYS read the source file and verify the fact still holds
2. If the fact changed -> update it via memory_write
3. NEVER make changes based on STALE facts without verification

## PROACTIVE MEMORY — IMPORTANT
When you discover something important during work, SAVE IT IMMEDIATELY:
- Bug and fix -> memory_write(type="error_fix")
- Pitfall/danger -> memory_write(type="gotcha")
- Exact procedure -> memory_write(type="procedure")
- How components communicate -> memory_write(type="api_contract")
- Performance issue -> memory_write(type="performance")
- Important command -> memory_write(type="command")
DO NOT wait for /harvest — session may crash.

## RUNNING BRIDGE — CRITICAL
After completing each task AUTOMATICALLY update session bridge:
  session_bridge(action="save", content="Progress: ...; Open: ...")
This is Tier 1 — do it yourself, don't announce, it's part of the job.

## Safety Rules — MANDATORY
- Before ANY deploy, push, ssh, pm2, docker, db migration:
  1. ALWAYS call verify_identity(action_type="...") first
  2. If it returns BLOCKED — STOP and ask the user
  3. If it returns VERIFIED — READ the target server to the user and request confirmation

## Git Rules
- Commit often, small atomic changes. Format: "[type] what and why"
- commit = Tier 1 (do it yourself). push = Tier 3 (verify_identity).

## Project DNA: Nexu
Stack: unknown
Style: [unknown]
Structure: .idea
Deploy: [NOT SET]
Active: [new session]
Last: [first session]

## Last Session Bridge
[Emergency bridge — running bridge was not updated]
Files: src/App/app.entitlements (create), src/App/Nexu.App.csproj (edit), src/App/Program.cs (edit)

# === END COGNILAYER ===

---
> Source: [Alpharius99/Nexu](https://github.com/Alpharius99/Nexu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
