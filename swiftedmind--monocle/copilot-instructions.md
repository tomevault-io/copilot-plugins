## monocle

> `monocle` is a read-only Swift code inspection tool built on top of **SourceKit-LSP**.

# AGENTS.md — Architecture & Contribution Guide

`monocle` is a read-only Swift code inspection tool built on top of **SourceKit-LSP**.  
Its primary purpose is to provide a command-line equivalent of “cmd-click” in an IDE: given a file, line, and column, it resolves the symbol there and returns information about its definition and documentation.

This document gives a high-level view of how the project is structured and how it is intended to evolve, so that contributors can extend it without breaking the overall design.

---

## General Instructions

Pay attention to these general instructions and closely follow them!

- Always prefer readability over conciseness/compactness.
- Never commit unless instructed to do so.

### When you are done

- Always build the project to check for compilation errors.
- When you have added or modified Swift files, always run `swiftformat --config ".swiftformat" {files}`.

## Build Instructions

- Build from the repository root with `swift build`.
- Prefer `swift build --quiet` to reduce noise; only drop `--quiet` when debugging a failure.

---

## 1. Purpose and Scope

- Inspect Swift symbols at a given file/line/column.
- Resolve definitions across the workspace, including packages and dependencies.
- Surface signatures and documentation via SourceKit-LSP.
- Operate in a read-only manner: no edits, no refactorings, no code generation.

The tool focuses on **understanding** existing Swift code, not modifying it.

---

## 2. High-Level Architecture

The project is structured into clear layers to keep responsibilities separate while allowing the daemon/server to build on the same foundations.

### 2.1 Core Layer

The core layer encapsulates all interaction with SourceKit-LSP and the workspace:

- Manages the lifecycle of an LSP session (starting and shutting down `sourcekit-lsp`).
- Sends LSP requests (e.g. “go to definition”, “hover”) and parses responses.
- Resolves the workspace (SwiftPM or Xcode) from file paths or an explicit root.
- Provides a small, stable API for “inspect the symbol at this location”.

This layer is intentionally unaware of command-line argument parsing, JSON printing, or daemon concerns.

### 2.2 CLI Layer

The CLI layer provides the `monocle` executable and user-facing interface:

- Parses arguments and options.
- Resolves the workspace using the core’s workspace utilities.
- Calls into the core to perform inspections.
- Formats results either as human-readable text or JSON.

The CLI is a thin wrapper around the core logic. Any substantial behavior should live in the core layer rather than in CLI commands.

### 2.3 Daemon Layer

The long-lived daemon/server keeps one or more LSP sessions alive across requests, handles JSON requests over a local socket (e.g. Unix domain socket), and manages idle timeouts and cleanup when sessions are unused.

The CLI includes commands to start and stop the daemon, and symbol-related commands prefer talking to the daemon when available, with a fallback to the “one LSP session per process” behavior in the core. The daemon builds on the same core API as the CLI.

---

## 6. Extension Guidelines

- New features should be evaluated in terms of which layer they belong to. LSP-related logic belongs in core, user interaction in the CLI, and connection management in the daemon.
- Changes to the core API should be made cautiously, as both CLI and daemon depend on it.
- When adding new commands or capabilities (e.g. references, document symbols), reuse the existing patterns:
  - Add an entry point in the core for the operation.
  - Expose it through CLI and, once implemented, through the daemon.
- Keep configuration surfaces small and explicit (flags, environment variables) and avoid hidden behavior.

---
> Source: [SwiftedMind/monocle](https://github.com/SwiftedMind/monocle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
