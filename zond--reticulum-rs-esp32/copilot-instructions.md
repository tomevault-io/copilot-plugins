## reticulum-rs-esp32

> Run the `date` command to understand what date it is today.

# Agent Instructions

## Development Workflow

Run the `date` command to understand what date it is today.

### Documentation ideals

README.md contains all project overview, build instructions, and basic know how necessary
to initially understand what the project is about.

CLAUDE.md contains all information an agent needs to work in the project that doesn't fit
in README.md.

docs/* contains specialized knowledge, TODO files, research results, future ideas, and
anything else that is documents but don't fit in README.md or CLAUDE.md.

README.md and CLAUDE.md should refer to docs/* files when useful.

### Before Every Commit

1. Run tests (both host and qemu, and if available esp32) and linting (see README.md for commands)
2. Format code: `cargo fmt`
3. Ensure README.md and CLAUDE.md are up to date
4. Check if any files in the docs directory need update
5. If available, review with code review agents (code-simplifier, rust-code-guardian)
6. Fix any high priority issues identified by code review before committing
7. Add low/medium priority improvements to docs/future-work.md or TODO comments

### Commit Standards

- Keep commits reasonably small and focused
- Each commit should be well-tested
- Write clear, descriptive commit messages
- Prefer many small commits over large monolithic ones
- Make sure that no code commited in the project refers to absolute paths, or paths
  outside the project directory. Documentation about how to install or manage deps
  may provide examples or instructions that refer to absolute paths or paths outside
  the project directory.
- Never use Rust unsafe code unless it's really necessary, and very well motivated
  in comments.

## Project Context

**Supported boards** (select via feature flag):
- `board-heltec-v4` - Heltec WiFi LoRa 32 V4 (ESP32-S3 + SX1262) - primary target
- `board-lilygo-t3` - LILYGO LoRa32 T3 V1.6.1 (ESP32 + SX1276)

Key constraints to keep in mind:
- Memory limits are board-specific (see `src/board/mod.rs`)
- Heltec V4: 512KB SRAM, 2MB PSRAM - use PSRAM for large buffers
- LILYGO T3: 520KB SRAM, PSRAM varies - conservative limits assumed
- ESP-IDF provides std environment (not bare metal no_std)
- BLE mesh is not upstream in reticulum-rs - needs custom implementation
- OLED display available on Heltec V4 (SSD1306, 128x64, I2C)

**Radio abstraction**: The `RadioHardware` trait (`src/lora/hardware.rs`) abstracts over different
LoRa chips. Board-specific implementations are in `src/lora/sx1262.rs` and `src/lora/sx1276.rs`.

For build commands, tool locations, and setup instructions, see [README.md](README.md).

## Local Device Configuration

The file `.local-devices` (gitignored) stores machine-specific device paths for two-device
LoRa testing. When this file exists, use the ports defined there:

```bash
# Source the file to get device configuration
source .local-devices

# Flash our firmware to the test device (DEVICE format: path,board)
DEVICE=$TEST_DEVICE cargo flash-esp32

# The RNode device runs official RNode firmware for Python Reticulum
```

Example `.local-devices` file:
```bash
# Local device configuration for two-device LoRa testing
RNODE_PORT=/dev/cu.usbserial-59680217731
TEST_DEVICE=/dev/cu.usbserial-59680249751,heltec-v4
# Or for LILYGO T3:
# TEST_DEVICE=/dev/cu.usbserial-59680249751,lilygo-t3
```

See [docs/lora-testing-strategy.md](docs/lora-testing-strategy.md) for the full two-device
testing setup.

## Key Documentation

| Document | Contents |
|----------|----------|
| [README.md](README.md) | Prerequisites, build commands, tool locations |
| [docs/qemu-setup.md](docs/qemu-setup.md) | QEMU installation and usage |
| [docs/research-findings.md](docs/research-findings.md) | Build configuration rationale |
| [docs/implementation-guide.md](docs/implementation-guide.md) | Feature implementation plans |
| [docs/testing-strategy.md](docs/testing-strategy.md) | Testing system, `#[esp32_test]` usage |
| [docs/lora-testing-strategy.md](docs/lora-testing-strategy.md) | Two-device LoRa testing with RNode |
| [docs/scalable-routing-proposal.md](docs/scalable-routing-proposal.md) | DHT-based routing for global scale |
| [docs/future-work.md](docs/future-work.md) | Planned improvements and TODOs |

---
> Source: [zond/reticulum-rs-esp32](https://github.com/zond/reticulum-rs-esp32) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
