## wire-seek

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Wire-seek is a WireGuard MTU optimization tool that discovers the optimal MTU for WireGuard tunnels using ICMP Path MTU Discovery with the Don't Fragment bit.

## Build and Development Commands

```bash
# Build the project
go build ./...

# Run tests
go test ./...

# Run a single test
go test -run TestName ./path/to/package

# Run tests with coverage
go test -cover ./...

# Format code
go fmt ./...

# Vet code for issues
go vet ./...
```

## Usage

```bash
# Basic usage (testing path to WireGuard endpoint, outside tunnel)
wire-seek <target-host>

# With options
wire-seek -v -target example.com   # Verbose mode
wire-seek -6 example.com           # Force IPv6
wire-seek -tunnel 8.8.8.8          # Testing THROUGH an existing tunnel
```

**Important**: Use `-tunnel` flag when testing through an existing WireGuard tunnel to avoid double-subtracting overhead.

## Architecture

- `main.go` - CLI entry point, argument parsing, and result display
- `mtu/` - MTU discovery package
  - `discover.go` - Binary search PMTUD algorithm using ICMP echo with DF bit
    - IPv4 range: 576-1500 bytes (RFC 791 minimum)
    - IPv6 range: 1280-1500 bytes (RFC 8200 minimum)
    - Verbose mode (`-v`) shows detailed socket and error diagnostics
  - `wireguard.go` - WireGuard overhead calculations (60 bytes IPv4, 80 bytes IPv6)
  - `df_linux.go`, `df_darwin.go`, `df_windows.go`, `df_other.go` - Platform-specific Don't Fragment bit handling

## Key Implementation Details

- **Linux**: Uses `IP_MTU_DISCOVER` socket option with `IP_PMTUDISC_DO` to set DF bit
- **Windows**: Uses `IP_DONTFRAGMENT` socket option
- **macOS/Darwin**: Uses control messages via `SetControlMessage`
- **Unprivileged ICMP**: Falls back to UDP-based ICMP on Linux 3.0+ when raw sockets unavailable

## Git Commit Guidelines

- Do NOT include `Co-Authored-By` lines in commit messages

---
> Source: [yeya/wire-seek](https://github.com/yeya/wire-seek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
