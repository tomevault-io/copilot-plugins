## msi-mpg-491cqp-control

> USB HID control for MSI MPG 491CQP monitor. Rust implementation.

# MSI MPG 491CQP Control

USB HID control for MSI MPG 491CQP monitor. Rust implementation.

## Project Structure

```text
crates/
├── msi-core/       # Core library (protocol, features, monitor API)
├── msi-cli/        # Command-line interface
├── msi-gui/        # Desktop GUI (Dioxus)
└── msi-hwtest/     # Hardware integration tests

docs/
├── protocol/       # Protocol specifications (source of truth)
├── guides/         # User guides
└── research/       # Research notes and data
```

## Before Making Changes

1. Run `cargo test` to verify tests pass
2. Read the relevant AGENTS.md in subdirectories (`crates/AGENTS.md`, `docs/AGENTS.md`)

## After Making Changes

1. Run `cargo fmt` (no approval needed)
2. Run `cargo clippy` and fix warnings
3. Run `cargo test` to verify nothing broke
4. For protocol changes, update `docs/protocol/usb-hid-reference.md`

## Always Do

- Commit changes after each logical step (don't wait to be reminded)
- When you discover a missing rule, add it to AGENTS.md
- Use Report ID `0x01` in all USB HID commands (0x00 fails silently)
- Encode feature codes as 5 ASCII characters (`0x0400` → `"00400"`)
- Use 3-digit decimal values for most features (`75` → `"075"`)
- Document protocol discoveries in `docs/protocol/`

## Never Do

- Use Report ID `0x00` (fails silently, no error returned)
- Commit binary files (pcap captures, firmware, etc.)
- Guess at protocol details without verification from captures
- Test unknown feature codes without asking (could have side effects)

## Ask First

- Testing unknown feature codes on hardware
- Adding new dependencies to Cargo.toml
- Major architectural changes

## Quick Commands

```bash
cargo build --release           # Build all
cargo test                # Unit tests
cargo run --bin msi-cli -- list # Test CLI
cargo clippy                    # Lint
just check                      # Full check (requires just)
```

## Hardware Setup

- **Linux:** Install udev rules: `sudo cp config/udev/99-msi-monitor.rules /etc/udev/rules.d/`
- **macOS:** No setup required
- **Windows:** HID driver usually automatic

## See Also

- `crates/AGENTS.md` - Rust code patterns
- `docs/AGENTS.md` - Documentation guidelines
- `docs/protocol/usb-hid-reference.md` - Protocol specification

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hamishmorgan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hamishmorgan)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
