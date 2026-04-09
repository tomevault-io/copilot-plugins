## cursor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a collection of Cursor configuration utilities and rules for Go-based projects.
It serves as a central hub for Cursor IDE configuration management through an automated initialization script.

## Key Scripts and Usage

### Initialization Script
- `init.sh`: Main initialization script that sets up Cursor configuration in Go projects
  - Clones this repository temporarily to copy configuration files
  - Creates `.cursor/` directory structure with rules and helper scripts
  - Vendors Go libraries to `.cursor/libraries/` for Cursor indexing
  - Updates `.gitignore` and `.cursorignore` appropriately
  - Usage: `curl -s https://raw.githubusercontent.com/erazemk/cursor/main/init.sh | sh`
  - Command line options: `-d` (debug), `-r` (no rules), `-f` (force), `-h` (help)

### Update Script
- `.cursor/update.sh`: Updates Cursor configuration by re-running the latest init script
  - Created by init.sh in target projects
  - Usage: `./cursor/update.sh` (from project root)

## Architecture

The initialization script creates this structure in target projects:

```
project/
├── .cursor/
│   ├── libraries/         # Vendored Go dependencies (git-ignored, cursor-indexed)
│   ├── rules/            # Cursor IDE rules (.mdc files)
│   │   ├── core.mdc      # General development guidelines  
│   │   ├── golang.mdc    # Go-specific best practices
│   │   └── libraries.mdc # Library usage patterns
│   ├── README.md         # Documentation for .cursor directory
│   └── update.sh         # Script to update configuration
```

### Library Vendoring Strategy
- Analyzes `go.mod` to identify direct dependencies
- Uses `go mod vendor` to download library source code  
- Copies only direct dependencies to `.cursor/libraries/`
- Allows Cursor IDE to index library code for better autocomplete and suggestions
- Addresses Cursor's limitation of not indexing directories with >10,000 files

### Rule System
- Core rules (`core.mdc`): General development best practices
- Go rules (`golang.mdc`): Go-specific conventions and error handling
- Library rules (`libraries.mdc`): Guidance for using specific libraries
- Rules use Cursor's `.mdc` format with glob patterns and metadata

## Development Commands

Since this is primarily a shell script project, testing involves:
- Manual testing with different Go projects
- Verifying script behavior with various command-line options
- Ensuring proper file creation and Git ignore patterns

## Go Project Context

The scripts are designed specifically for Go projects:
- Requires `go.mod` file for dependency analysis
- Uses `go mod vendor` for library downloading
- Follows Go project conventions and standards
- Integrates with Go toolchain (`go vet`, `go test`, `go mod tidy`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erazemk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/erazemk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
