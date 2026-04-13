## tools-installer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive toolkit installer for Debian 9 systems that builds essential development tools without root access. The installer downloads, compiles, and installs tools to `~/tools/` with automatic dependency resolution, parallel compilation, and rollback capabilities.

## Common Commands

### Main Operations
```bash
./installer help          # Show help and available commands
./installer list          # List all available tools and their status
./installer download all  # Download all tool sources without building
./installer install all   # Install all tools in phases
./installer status        # Check installation status of all tools
./installer test [TOOL]   # Test if a specific tool is working
./installer uninstall TOOL  # Remove a specific tool
./installer clean TARGET  # Clean specified directories (downloads/builds/logs/all)
```

### Download and Installation Targets
```bash
./installer download phase1    # Download Phase 1 sources (cmake, ninja, python)
./installer download phase2    # Download Phase 2 sources (gcc, make, openssl, etc.)
./installer download phase3    # Download Phase 3 sources (zlib, libffi, etc.)
./installer download cmake     # Download specific tool source

./installer install phase1    # Pre-compiled binaries (cmake, ninja, python)
./installer install phase2    # Build tools (gcc, make, openssl, etc.)
./installer install phase3    # Essential libraries (zlib, libffi, etc.)
./installer install cmake     # Install specific tool
```

### Version Override (Download Only)
```bash
./installer download cmake --version 3.25.0      # Download specific cmake version
./installer download gcc --version 11.4.0        # Download specific gcc version
./installer download phase1 --version 3.26.0     # Apply version to all Phase 1 tools (not recommended)
```

### Cleaning
```bash
./installer clean downloads   # Remove downloaded source files
./installer clean builds      # Remove temporary build directories  
./installer clean logs        # Remove log files
./installer clean all         # Remove all temporary files
```

### Debugging
```bash
./installer download all --verbose # Show detailed download progress
./installer install all --verbose  # Show detailed command output
tail -f logs/cmake-download.log       # Monitor cmake download progress
tail -f logs/gcc-build.log            # Monitor gcc build progress
tail -f logs/debug.log                # View debug information
ls logs/                              # List all per-tool log files
```

## Architecture

### Directory Structure
- `installer` - Main entry point and command dispatcher
- `config/` - Configuration files (versions.conf, mirrors.conf)
- `lib/` - Reusable library functions (common, build, download, test)
- `installers/` - Individual tool installation scripts
- `downloads/` - Downloaded source tarballs (created during install)
- `build/` - Temporary build directories (created during install)
- `logs/` - Per-tool installation, download, and build logs (created during install)

### Key Components

#### Configuration System
- `config/versions.conf` - Tool versions and download URLs
- `config/mirrors.conf` - Mirror URLs for speed testing

#### Library Functions
- `lib/common.sh` - Logging, disk space checks, CPU detection
- `lib/build.sh` - Build functions (configure, make, cmake)
- `lib/download.sh` - Download with mirror selection and checksums
- `lib/test.sh` - Tool verification functions

#### Installation Phases
1. **Phase 1**: Pre-compiled binaries (cmake, ninja, python)
2. **Phase 2**: Build from source tools (gcc, binutils, make, openssl, etc.)  
3. **Phase 3**: Essential libraries (zlib, libffi, readline, etc.)

### Build System
- Uses parallel compilation (`-j$(nproc)`)
- Installs to versioned directories (e.g., `~/tools/gcc-10.5.0/`)
- Creates symlinks for current versions (e.g., `~/tools/gcc -> gcc-10.5.0`)
- Comprehensive per-tool logging (e.g., `logs/gcc-build.log`, `logs/cmake-download.log`) and `logs/debug.log`

### Tool Testing
Each tool has verification tests in `lib/test.sh` that check basic functionality:
```bash
./installer test gcc    # Test GCC installation
./installer test all    # Test all installed tools
```

## Environment Setup

Tools are installed to `~/tools/` with the following structure:
```
~/tools/
├── cmake -> cmake-3.27.9/
├── gcc -> gcc-10.5.0/
├── python -> python-3.11.8/
└── ...
```

To use installed tools, add to PATH:
```bash
export PATH="$HOME/tools/cmake/bin:$HOME/tools/gcc/bin:$HOME/tools/python/bin:$PATH"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rojosinalma)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rojosinalma)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
