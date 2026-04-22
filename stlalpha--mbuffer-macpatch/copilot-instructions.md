## mbuffer-macpatch

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

mbuffer is a measuring buffer tool written in C that provides high-performance data buffering with features like:
- Memory-mapped I/O support for large buffers
- Multi-threaded architecture using pthreads and semaphores
- Network support (TCP/IP) for data transfer
- Multi-volume and autoloader support for tape devices
- Hash computation (MD5) capabilities
- I/O speed measurement and display

## Build System

This project supports two build systems:

### Autoconf/Make (Primary)
```bash
# Configure and build
./configure
make

# Build with specific flags (e.g., 64-bit)
env CFLAGS="-O -g -m64" ./configure
make

# Run tests
make check

# Install
make install

# Clean
make clean
make distclean
```

### CMake (Alternative)
```bash
# Configure and build
cmake .
cmake --build .

# Run tests (if ENABLE_TESTING is set)
ctest
```

## Testing

Tests are located in the `tests/` directory (test0.sh through test7.sh). The test suite includes:
- Basic buffering tests (test0, test1)
- IPv4 networking tests (test2)
- IPv6 networking tests (test3)
- Multi-volume tests (test4, test5)
- Device I/O tests (test6)
- Hash computation tests (test7)

Tests use tar archives and MD5 checksums for verification. Some tests require external tools:
- `openssl` for MD5 computation
- `have-af` utility to check IPv4/IPv6 support
- LD_PRELOAD libraries (`idev.so`, `tapetest.so`) for simulating device behavior

## Code Architecture

### Core Components

**mbuffer.c** (main program)
- Main entry point and orchestration
- Signal handling (SIGINT, SIGHUP)
- Thread management for input, output, status, and watchdog
- Buffer management using semaphores (Buf2Dev, Dev2Buf)
- Volume management for multi-tape operations

**input.c/input.h**
- Input thread implementation
- Handles reading from files, devices, or network
- Speed limiting for input operations
- Multi-volume input support

**network.c/network.h**
- TCP/IP networking implementation
- IPv4 and IPv6 support
- Server and client socket handling

**hashing.c/hashing.h**
- Hash computation (MD5 and other algorithms)
- Runs in separate threads as "hashers"

**log.c/log.h**
- Logging infrastructure with multiple severity levels
- Status message formatting

**settings.c/settings.h**
- Command-line option parsing
- Configuration file reading
- Parameter validation

**globals.c/globals.h**
- Global variable declarations
- Shared state between threads

**common.c/common.h**
- Utility functions
- Buffer initialization
- Direct I/O handling

### Threading Model

The application uses a multi-threaded producer-consumer pattern:
- **Input thread** (`inputThread`): Reads data into circular buffer
- **Output thread** (`outputThread`): Main coordinator, writes to primary output
- **Sender threads** (`senderThread`): Write to additional outputs in parallel
- **Hash threads** (`hashThread`): Compute checksums
- **Status thread** (`statusThread`): Display progress statistics
- **Watchdog thread** (`watchdogThread`): Monitor for stalls

Synchronization uses:
- Semaphores: `Buf2Dev` (blocks ready for output), `Dev2Buf` (blocks available for input)
- Mutexes: `SendMut`, `TermMut`, `LowMut`, `HighMut`
- Condition variables: `SendCond`, `PercLow`, `PercHigh`

### Buffer Management

- Circular buffer array (`Buffer[]`) with configurable size
- Block-based I/O with tunable block size (`Blocksize`)
- Watermark system for flow control (`StartRead`, `StartWrite`)
- Optional memory-mapped files for very large buffers

## Platform Notes

- **Linux**: Fully supported (primary platform)
- **Solaris**: Fully supported
- **FreeBSD**: Supported with some caveats
- **macOS**: Not supported due to missing `sem_getvalue()` - the README explicitly states this limitation
- Requires POSIX threads, semaphores, and optionally `sendfile()` for zero-copy I/O

## Key Configuration

Configuration files are read in order:
1. `/etc/mbuffer.rc`
2. `${SYSCONFDIR}/mbuffer.rc`
3. `~/.mbuffer.rc`
4. File specified in `$MBUFFERRC` environment variable

Version generation uses `mkversion.sh` script to create `version.h`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stlalpha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
