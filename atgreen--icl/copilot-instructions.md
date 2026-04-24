## icl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Install dependencies (requires ocicl package manager)
ocicl install

# Build the icl executable
make

# Clean build artifacts
make clean
```

The build requires SBCL, [ocicl](https://github.com/ocicl/ocicl), and libfixposix-devel.

## Running

```bash
# Start ICL with auto-detected Lisp backend
./icl

# Specify a backend
./icl --lisp sbcl
./icl --lisp ccl

# Evaluate and exit
./icl -e '(+ 1 2 3)'

# Connect to existing Slynk server
./icl --connect localhost:4005
```

## Architecture Overview

ICL is a frontend REPL that communicates with a backend Lisp process via the Slynk protocol (from SLY). This client-server architecture allows it to work with multiple Lisp implementations.

### Key Components

- **src/main.lisp** - Entry point, CLI parsing (uses clingon)
- **src/repl.lisp** - Main REPL loop, coordinates input/output/eval
- **src/slynk-client.lisp** - Wraps the slynk-client library for backend communication
- **src/backend.lisp** - Backend process management, Lisp implementation detection
- **src/editor.lisp** - Multi-line input editor with readline-style editing
- **src/buffer.lisp** - Input buffer management for the editor
- **src/paredit.lisp** - Structural editing (auto-close parens, sexp navigation)
- **src/completion.lisp** - Tab completion for symbols and packages
- **src/highlight.lisp** - Syntax highlighting with terminal colors
- **src/inspector.lisp** - TUI object inspector
- **src/mcp-server.lisp** - HTTP server for AI CLI integration (read-only tools)
- **src/browser.lisp** - Web-based IDE interface (Hunchentoot + WebSockets)
- **src/commands/** - Extensible comma-prefixed command system

### Command System

Commands are defined using `define-command` in `src/commands/core.lisp`:

```lisp
(define-command (help h ?) ()
  "Show available commands."
  ...)
```

Commands can have aliases (like `h` and `?` above) and receive parsed arguments as strings.

### Terminal Abstraction

Platform-specific terminal handling:
- `src/terminal-posix.lisp` - POSIX termios-based raw mode
- `src/terminal-windows.lisp` - Windows console API

### Dependencies

Third-party code is vendored in:
- `3rd-party/slynk-client/` - Modified slynk-client for backend communication
- `ocicl/` - Dependencies managed by ocicl (including Slynk from SLY)

## Release Process

See `docs/RELEASING.md`. Key files to update:
- `icl.asd` - `:version` field
- Create `docs/release-notes/RELEASE-NOTES-X.Y.Z.md`

Tag with `vX.Y.Z` to trigger GitHub Actions build.

---
> Source: [atgreen/icl](https://github.com/atgreen/icl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
