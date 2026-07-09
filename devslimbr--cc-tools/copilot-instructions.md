## cc-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Note: The agent that interacts with this repository must communicate in English in all its interactions.

## Project Overview

CCTools is a CLI tool in Go for file editing that **automatically preserves the original encoding** of files during operations. It is especially useful for working with legacy code, international projects, and files with different character encodings.

## Build and Development Commands

### Build for current platform:
```bash
go build -o cctools
```

### Cross-platform build:
```bash
chmod +x build.sh
./build.sh
```

### Dependency management:
```bash
go mod tidy
```

### Direct execution:
```bash
go run main.go [command] [flags]
```

## Code Architecture

### Main structure:
- **`main.go`**: Entry point that delegates to cmd.Execute()
- **`cmd/`**: CLI commands using Cobra framework
  - `root.go`: Root command and global configuration
  - `read.go`, `write.go`, `edit.go`, `multiedit.go`: Implementation of main commands
- **`pkg/encoding/`**: Character encoding detection and conversion
- **`pkg/fileops/`**: File operations with support for multiple encodings
- **`internal/models/`**: Shared data structures

### Operation flow:
1. **Detection**: `encoding.Detector` identifies the file encoding using chardet
2. **Operation**: `fileops.FileOperations` executes the operation preserving encoding
3. **Security**: Automatic backup and rollback on failure
4. **Atomicity**: MultiEdit ensures that all operations succeed or fail together

### Supported encodings:
- UTF-8, UTF-16 (LE/BE)
- ISO-8859-1, ISO-8859-15
- Windows-1252, Windows-1251
- GB18030, GBK, Big5
- Shift_JIS, EUC-JP, EUC-KR

## Available Commands

### cctools read
Reads files with automatic encoding detection:
```bash
./cctools read --file file.txt [--detect-encoding] [--verbose]
```

### cctools write
Creates/overwrites files with specified encoding:
```bash
./cctools write --file file.txt --content "content" [--encoding UTF-8]
```

### cctools edit
Edits files preserving original encoding:
```bash
./cctools edit --file file.txt --old "old text" --new "new text" [--replace-all]
```

### cctools multiedit
Multiple atomic edits through JSON file:
```bash
./cctools multiedit --edits-file operations.json
```

## Development Standards

### When working with this code:
1. **Always use encoding-aware operations**: The project was created specifically to preserve encodings
2. **Test with files of different encodings**: Check compatibility with ISO-8859-1, Windows-1252, etc.
3. **Maintain atomic operations**: Especially important in MultiEdit
4. **Preserve layered architecture**:
   - CLI commands (cmd/) → File operations (pkg/fileops/) → Encoding handling (pkg/encoding/)

### Testing approach:
- Test with files of different encodings in the `test_files/` folder (if it exists)
- Verify that encoding is preserved after edits
- Test rollback in failure scenarios

### Cross-platform considerations:
- build.sh generates binaries for Windows, Linux, macOS and FreeBSD
- Line endings are preserved according to the operating system
- File paths must be handled appropriately on all platforms

## Technical Documentation

Refer to `docs/tools.md` for technical details of editing tools and `docs/prompt.md` for specific instructions on using CCTools tools.

---
> Source: [devslimbr/cc-tools](https://github.com/devslimbr/cc-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
