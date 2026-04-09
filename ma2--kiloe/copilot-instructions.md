## kiloe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a terminal-based text editor written in C, similar to the "kilo" editor. It's a single-file implementation that runs in raw terminal mode with features like syntax highlighting, search, and file operations.

## Build and Run Commands

```bash
# Build the text editor
make

# Run the editor
./kiloe [filename]

# Clean build
rm -f kiloe
make
```

## Architecture

### Core Implementation (`kiloe.c`)

The entire editor is implemented in a single C file with the following major components:

1. **Terminal Management Layer** - Raw mode handling, escape sequences, terminal I/O
2. **Data Structures** - Editor state (`editorConfig`), row management (`erow`), append buffers
3. **Input Processing** - Keyboard event handling, special key mapping, command processing
4. **Rendering System** - Screen refresh logic, syntax highlighting engine, status bar
5. **File Operations** - File reading/writing with dirty state tracking

### Key Data Structures

- `struct editorConfig` - Global editor state including cursor position, file content, terminal dimensions
- `struct erow` - Individual text rows with render state and syntax highlighting
- `struct editorSyntax` - Syntax highlighting rules for different file types

### Important Functions

- `editorOpen()` - Opens and loads a file into the editor
- `editorSave()` - Saves the current buffer to file
- `editorProcessKeypress()` - Main input handling loop
- `editorRefreshScreen()` - Redraws the terminal display
- `editorFind()` - Implements search functionality

## Development Notes

- Uses POSIX terminal APIs (`termios`, `unistd.h`)
- Requires C99 standard (`-std=c99`)
- No external dependencies beyond standard C library
- Syntax highlighting currently supports C/C++ files
- Terminal escape sequences are used for cursor control and colors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ma2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ma2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
