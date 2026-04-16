## lxc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LXC Tools is a collection of Bash scripts that simplify LXC (Linux Container) management operations. The project provides command-line utilities for creating, managing, and maintaining LXC containers with streamlined workflows.

## Architecture

The project follows a simple script-based architecture where each tool is a standalone Bash script:

- **Core Tools**: Simple wrapper scripts around LXC commands with intelligent defaults
- **Network Management**: Scripts for setting up macvlan networking profiles
- **SSH Integration**: Automatic authorized_keys deployment for container access
- **Container Lifecycle**: Complete container management from creation to destruction

## Commands

### Installation
```bash
# Install all tools to system PATH
./install
```

### Container Management
```bash
# List all containers
lx

# Create container with macvlan profile
newlx <os> <name>

# Create container with default profile + SSH keys
newdlx <os> <name>

# Connect to container
conlx <container>

# Delete container
rmlx <container>

# Clean restart container (preserves original OS/version)
redolx <container>

# Clean restart with SSH key deployment
redodlx <container>
```

### Network Configuration
```bash
# Setup macvlan networking profile for LAN IP addresses
setupNetwork
```

### Utilities
```bash
# List available images
lstImages

# Deploy SSH authorized_keys to container
authkey <container>

# Create port forward rule
pflx <container> <name>
```

## Code Conventions

- All scripts use `#!/bin/bash` or `#!/usr/bin/bash` shebangs
- Scripts follow camelCase naming convention
- Use `set -x` for debugging in development scripts (newdlx, redodlx)
- Exit codes: 0 for success, 1 for errors
- No external dependencies beyond LXC and standard Unix tools

## Development Workflow

1. **Testing**: Scripts should be tested manually with actual LXC containers
2. **Error Handling**: Check for LXC installation before proceeding (`which lxc`)
3. **Shell Compatibility**: Scripts use Bash-specific features, ensure bash is available
4. **Permissions**: Installation requires sudo for copying to /usr/local/bin/

## Key Implementation Details

- **Network Setup**: Automatically detects default network interface for macvlan
- **SSH Integration**: Uses ~/.ssh/authorized_keys for container access
- **Container Refresh**: redolx/redodlx preserve original image configuration from container metadata
- **Port Forwarding**: pflx creates TCP proxy devices with sensible defaults (80→127.0.0.1:80)

## Build and Release

- **CI/CD**: GitHub Actions workflow builds Debian packages on version tags (v*.*.*) 
- **Package**: Builds .deb packages for amd64 architecture
- **Distribution**: Installs scripts to /usr/local/bin/ system-wide

## Container Image Sources

Scripts expect standard LXC image formats:
- `ubuntu:20.04`, `ubuntu:22.04`
- `debian:11`, `debian:12` 
- Other images available via `lstImages`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Centralize) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
