## claudebox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is **claudebox** - a macOS sandbox wrapper for Claude Code that provides secure, isolated execution environments. The repository contains a single main bash script that wraps `sandbox-exec` to run Claude Code with restricted filesystem and system access.

## Key Components

- **`claudebox`** - Main bash script (v0.1) that creates dynamic sandbox profiles
- **`install.sh`** - Installation script for system-wide deployment
- **`README.md`** - Comprehensive documentation and usage guide
- **`logo.svg`** - Project logo

## Architecture

### Core Functionality
The `claudebox` script operates through several key phases:

1. **Environment Detection** (`detect_package_paths`) - Automatically discovers package managers:
   - Homebrew (ARM: `/opt/homebrew`, Intel: `/usr/local/Homebrew`)
   - User binaries (`~/.local`)
   - Node version managers (`~/.nvm`, `~/.fnm`, `~/.nodenv`)
   - Nix (`/nix/store`)

2. **Profile Generation** (`generate_profile`) - Creates dynamic sandbox profiles:
   - Base profile template with security restrictions
   - Path injection for detected package managers
   - Project-specific access permissions

3. **Sandbox Execution** (`run_claude`) - Executes Claude Code within sandbox:
   - Profile validation using `sandbox-exec -f profile true`
   - Process limits via `ulimit -u`
   - Full argument passthrough to Claude Code

### Security Model
- **Default deny** policy with explicit allow rules
- **Project directory access** - Full read/write/execute for current working directory
- **System paths** - Read-only access to `/usr`, `/bin`, `/sbin`, `/System`
- **Blocked paths** - Personal directories, SSH keys, AWS credentials, etc.
- **Network access** - Unrestricted (external firewall recommended)

## Development Commands

### Basic Usage
```bash
# Run Claude Code in sandbox (default behavior)
./claudebox

# Pass arguments to Claude Code
./claudebox run --help
./claudebox run --dangerously-skip-permissions
```

### Development & Debugging
```bash
# Enable verbose debug logging
CLAUDEBOX_VERBOSE=1 ./claudebox

# Dry run mode (show what would be executed)
CLAUDEBOX_DRY_RUN=1 ./claudebox

# Generate sandbox profile only
./claudebox generate

# Validate generated profile
./claudebox validate

# Show profile file path
./claudebox profile
```

### Installation
```bash
# Install to ~/.local/bin
./install.sh

# System-wide installation
sudo cp claudebox /usr/local/bin/
```

## Configuration

### Environment Variables
- `CLAUDEBOX_VERBOSE=1` - Enable debug output
- `CLAUDEBOX_DRY_RUN=1` - Preview mode without execution
- `CLAUDEBOX_CONFIG=path` - Custom config file location

### Config File (`~/.claudeboxrc`)
```bash
# Example configuration
ULIMIT_PROCESSES=2048
VERBOSE=1
```

## Key Functions

- `detect_package_paths()` - Package manager discovery with 1-hour caching
- `create_base_profile()` - Generates sandbox profile template  
- `generate_profile()` - Creates complete project-specific profile
- `validate_profile()` - Tests profile syntax with sandbox-exec
- `run_claude()` - Main execution function with full workflow

## Testing

```bash
# Test profile generation
./claudebox generate

# Validate generated profile
./claudebox validate

# Test in dry-run mode
CLAUDEBOX_DRY_RUN=1 ./claudebox

# Debug package detection
CLAUDEBOX_VERBOSE=1 ./claudebox generate
```

## Important Notes

- **macOS only** - Requires `sandbox-exec` utility (not available on Linux/Windows)
- **Package detection caching** - Results cached for 1 hour in `$TMPDIR/.claudebox-paths-$(whoami)`
- **Profile cleanup** - Temporary profiles automatically removed on exit
- **Error handling** - Comprehensive error trapping with line numbers and function context

---
> Source: [Greitas-Kodas/claudebox](https://github.com/Greitas-Kodas/claudebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
