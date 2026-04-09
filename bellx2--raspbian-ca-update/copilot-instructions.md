## raspbian-ca-update

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go utility for updating CA certificates on old Raspbian systems where package management cannot update ca-certificates due to OpenSSL version dependencies. The tool downloads the latest CA bundle from curl.se and updates the system certificates.

## Build Commands

```bash
# Build the binary
go build -o raspbian-ca-update raspbian_ca_update.go

# Install to system (requires sudo)
sudo cp raspbian-ca-update /usr/local/bin/
```

## Architecture and Key Components

The tool is a single-file Go application with the following core functionality:

1. **System Validation**: Checks if running on Raspbian and validates CA certificate paths. Can be bypassed with `--force` flag.

2. **Certificate Update Process**:
   - Creates backup at `/etc/ssl/certs/ca-certificates.crt.backup`
   - Downloads from `https://curl.se/ca/cacert.pem`
   - Sets proper permissions (644)
   - Rebuilds certificate hash links using `c_rehash` or manual linking

3. **Safety Features**:
   - Requires root privileges
   - Creates backup before modifications
   - Tests SSL connection after update
   - Provides rollback capability via backup

## Command Line Options

- `--help`, `-h`: Show help message
- `--version`, `-v`: Show version information
- `--check`: Check current CA certificates status without updating
- `--force`: Force execution on non-Raspbian systems

## Development Notes

- Go 1.24.4 or later required
- No external dependencies - uses only Go standard library
- All text output and comments should be in English for global distribution
- Git commit messages should be in English with appropriate prefixes
- When modifying, ensure backward compatibility with old Raspbian systems
- The tool must handle systems where `c_rehash` might not work properly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bellx2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bellx2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
