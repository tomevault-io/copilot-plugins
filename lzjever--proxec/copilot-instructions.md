## proxec

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

proxec is a transparent TCP proxy executor that intercepts `connect()` syscalls via ptrace and routes connections through HTTP/SOCKS5 proxies. It works with any program without requiring LD_PRELOAD, making it compatible with setuid binaries.

## Build Commands

```bash
make build          # Debug build
make release        # Optimized build (LTO, stripped)
make test           # Run tests
make lint           # Run clippy (warnings as errors)
make fmt            # Format code
make fmt-check      # Check formatting
```

For development iterations:
```bash
cargo test          # Run all tests
cargo test --lib test_name  # Run specific test
cargo clippy -- -D warnings # Lint with warnings as errors
```

## Architecture

The program uses ptrace to intercept syscalls in a child process:

1. **Main thread** forks a child process with `PTRACE_TRACEME`, then execs the target program
2. **Tracer** intercepts `socket()` and `connect()` syscalls, tracking socket state and redirecting connections through the proxy
3. **Proxy handler** (async via tokio) establishes connections through HTTP CONNECT or SOCKS5 protocols

### Module Responsibilities

| Module | Purpose |
|--------|---------|
| `cli` | CLI argument parsing (clap) |
| `env` | Parse proxy env vars (`http_proxy`, `all_proxy`, `no_proxy`) |
| `tracer` | ptrace syscall interception, architecture-specific register/memory access |
| `proxy` | HTTP CONNECT and SOCKS5 protocol implementations |
| `socket` | Socket state tracking (pid+fd → destination mapping) |
| `error` | Error types with POSIX exit codes |

### Key Implementation Details

- **Architecture abstraction**: `src/tracer/arch/` has x86_64 and aarch64 implementations for syscall number/argument extraction
- **Memory access**: Uses `process_vm_readv`/`process_vm_writev` for reading child's sockaddr structures
- **IPv6**: Disabled by default for reliability; enable via `-6` flag or `PROXEC_IPV6=1`
- **no_proxy**: Supports exact hostnames, domain suffixes (`.example.com`), IP addresses, and CIDR notation

## Proxy Environment Variables

Standard Unix convention (lowercase takes precedence):
- `http_proxy` / `HTTP_PROXY` - HTTP proxy
- `https_proxy` / `HTTPS_PROXY` - HTTPS proxy
- `all_proxy` / `ALL_PROXY` - Proxy for all protocols
- `no_proxy` / `NO_PROXY` - Bypass list (comma-separated)

## Code Style

- Run `make fmt` before committing
- All clippy warnings must be fixed (`make lint`)
- Follow conventional commits for commit messages
- GPL-2.0-only license

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lzjever) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
