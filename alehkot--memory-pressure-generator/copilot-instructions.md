## memory-pressure-generator

> This is a C-based command-line tool designed to generate memory pressure on Android devices, primarily for testing Low Memory Killer (LMK) behavior in Android applications during development.

# Memory Pressure Generator

## Project Overview

This is a C-based command-line tool designed to generate memory pressure on Android devices, primarily for testing Low Memory Killer (LMK) behavior in Android applications during development.

## Purpose

The tool helps developers:
- Trigger LMK crashes intentionally for testing
- Test app behavior under memory pressure conditions
- Debug memory-related issues in Android applications

## Technology Stack

- **Language**: C
- **Platform**: Android (compiled with Android NDK)
- **Target**: Android devices/emulators

## Project Structure

- `memory_pressure_generator.c` - Main source file containing the memory allocation logic
- `README.md` - Build and deployment instructions
- `LICENSE` - Project license

## Key Functionality

The tool allocates memory gradually using multiple processes:
- Spawns multiple child processes (max 1000 MB per process)
- Allocates memory in configurable steps with delays
- Supports pause/resume via 'p' key during execution
- Uses SIGUSR1 for inter-process communication

## Build Process

Requires Android NDK toolchain. The build architecture must match the target emulator/device.

### Architecture Support

**ARM64 (aarch64):**
- Use for: ARM64 emulators on M1/M2/M3 Macs, ARM64 physical devices
- Compiler: `aarch64-linux-android30-clang`
- Common on modern Apple Silicon Macs

**x86_64:**
- Use for: x86_64 emulators on Intel Macs/PCs
- Compiler: `x86_64-linux-android30-clang`
- Older emulators or Intel-based systems

### Build Examples

**For ARM64 (Mac M1/M2/M3):**
```bash
/Users/alehkot/Library/Android/sdk/ndk/26.2.11394342/toolchains/llvm/prebuilt/darwin-x86_64/bin/aarch64-linux-android30-clang memory_pressure_generator.c -o memory_pressure_generator
```

**For x86_64 (Intel emulators):**
```bash
/Users/alehkot/Library/Android/sdk/ndk/26.2.11394342/toolchains/llvm/prebuilt/darwin-x86_64/bin/x86_64-linux-android30-clang memory_pressure_generator.c -o memory_pressure_generator
```

**Automated build (recommended):**
```bash
./build_and_deploy.sh  # Auto-detects architecture
./build_and_deploy.sh . arm64  # Force ARM64
./build_and_deploy.sh . x86_64  # Force x86_64
```

## Deployment

Push to Android device/emulator:
```bash
adb push memory_pressure_generator /data/local/tmp/
```

## Usage

```bash
./memory_pressure_generator <total_memory_in_mb> <steps> <delay_ms>
```

- `total_memory_in_mb`: Total memory to allocate across all processes
- `steps`: Number of allocation steps
- `delay_ms`: Delay between allocation steps in milliseconds

## Code Architecture

### Original Version (memory_pressure_generator.c)
- `allocate_memory()`: Core function that allocates memory gradually in steps
- `signalHandler()`: Handles SIGUSR1 for pause/resume functionality
- `enableRawMode()`/`disableRawMode()`: Terminal control for interactive input
- Multi-process architecture: Spawns processes via fork() when total memory exceeds MAX_MEMORY_PER_PROCESS

### Enhanced Version (memory_pressure_generator_enhanced.c)
- All features from original version plus:
- `read_meminfo()`: Parses /proc/meminfo for memory status tracking
- `read_psi_memory()`: Parses /proc/pressure/memory for PSI metrics
- `print_memory_status()`: Comprehensive real-time memory and pressure reporting
- mlock() support: Prevents swapping with `--use-mlock` flag
- Active page retention: Periodically touches pages to prevent eviction
- Process death detection: Monitors and reports LMK kills
- Automatic status updates: Shows status every 5 seconds
- Enhanced allocation: Writes to pages multiple times to ensure they're dirty and non-reclaimable

## Development Notes

### Original Implementation
- Maximum memory per process: 1000 MB (defined by MAX_MEMORY_PER_PROCESS)
- Memory allocation pattern: 0xAA bytes
- Processes stay alive after allocation completes
- Parent process manages all children and handles termination

### Enhanced Implementation
- Same multi-process architecture
- Adds PSI monitoring for Android 10+ (requires CONFIG_PSI=y kernel)
- Uses mlock() to prevent kernel from swapping pages
- Actively maintains memory pressure by touching pages every 5 seconds
- Detects LMK activity by monitoring child process termination
- Provides real-time feedback on memory pressure state
- More reliable LMK triggering on emulators with swap/ZRAM

### Why Enhanced Version Guarantees LMK on Emulators

**Problem with basic approach:**
1. Emulators often have generous RAM (4GB+) and swap/ZRAM enabled
2. Linux memory overcommit can delay physical allocation
3. Kernel may swap pages instead of triggering LMK
4. No feedback loop to verify pressure is building

**Enhanced version solutions:**
1. Forces physical memory allocation with memset() and multiple page writes
2. Uses mlock() to prevent swapping (when --use-mlock is used)
3. Monitors PSI to verify memory pressure is building
4. Tracks MemAvailable to ensure pressure is sustained
5. Actively touches pages to prevent page eviction
6. Detects and reports when processes are killed by LMK

**Android LMK thresholds (default):**
- Partial stall: 70ms (triggers at ~7% avg10 in PSI)
- Full stall: 700ms (triggers at ~70% avg10 in PSI)

## Helper Scripts

### build_and_deploy.sh
- Auto-detects Android NDK installation
- Builds both original and enhanced versions
- Deploys to connected device/emulator
- Sets proper permissions

### test_lmk_trigger.sh
- Analyzes device memory configuration
- Calculates optimal allocation parameters
- Provides preset configurations (conservative/aggressive)
- Monitors LMK activity in real-time via logcat
- Interactive test runner

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alehkot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alehkot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
