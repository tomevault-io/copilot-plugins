## uuider

> Whenever changes are made to this project, append an entry to `agent-log.md` with:

# Agent Instructions

## Change Log Requirement

Whenever changes are made to this project, append an entry to `agent-log.md` with:
- A timestamp (date, or date and time if multiple changes on the same day)
- A brief description of what was changed

## Rust Edition

This project uses Rust edition 2024. Do not change the edition to 2021 or earlier - edition 2024 is valid and intentional.

## Project Overview

`uuider` is a CLI tool for generating UUIDs, specifically UUIDv4 and UUIDv7. It allows generating random UUIDs or deterministic UUIDs from hex constants.

## Dependencies

- `clap` (v4, with derive feature) - CLI argument parsing
- `rand` (v0.8) - Random number generation
- `chrono` (v0.4) - Listed but not currently used; `std::time::SystemTime` is used instead

## CLI Commands

### `v4` - Generate UUIDv4
```
uuider v4 [--from-hex <HEX>]
```
- Without flags: generates a random UUIDv4
- `--from-hex <HEX>`: uses the provided hex value (up to 122 bits) as the UUID content

### `v7` - Generate UUIDv7
```
uuider v7 [--time <MILLIS>] [--from-hex <HEX> | --min | --max]
```
- Without flags: generates UUIDv7 with current timestamp and random data
- `--time <MILLIS>`: uses specific timestamp (milliseconds since Unix epoch)
- `--from-hex <HEX>`: uses hex value (up to 74 bits) for the random portion
- `--min`: sets all random bits to zero (74 bits = 0)
- `--max`: sets all random bits to one (74 bits = max)

Note: `--from-hex`, `--min`, and `--max` are mutually exclusive. All can be combined with `--time`.

## Code Structure (src/main.rs)

### Core Functions

| Function | Description |
|----------|-------------|
| `format_uuid(bytes)` | Formats 16 bytes into standard UUID string (8-4-4-4-12) |
| `parse_hex_to_bytes(hex)` | Parses hex string (supports `0x` prefix) to byte vector |
| `generate_v4_random()` | Generates random UUIDv4 |
| `generate_v4_from_hex(hex)` | Generates UUIDv4 from 122-bit hex constant |
| `generate_v7_now_random()` | Generates UUIDv7 with current timestamp |
| `generate_v7_with_time_random(timestamp)` | Generates UUIDv7 with specified timestamp |
| `generate_v7_with_time_min(timestamp)` | Generates UUIDv7 with specified timestamp and all random bits set to 0 |
| `generate_v7_with_time_max(timestamp)` | Generates UUIDv7 with specified timestamp and all random bits set to 1 |
| `generate_v7_with_time_hex(timestamp, hex)` | Generates UUIDv7 with specified timestamp and 74-bit hex constant |

### UUID Bit Layout

**UUIDv4 (RFC 4122):**
- 122 bits: random/constant data
- 4 bits: version (`0100` = 4) at bits 48-51 (byte 6 upper nibble)
- 2 bits: variant (`10`) at bits 64-65 (byte 8 upper 2 bits)

**UUIDv7 (RFC 9562):**
- 48 bits: Unix timestamp in milliseconds (bytes 0-5)
- 4 bits: version (`0111` = 7) at bits 48-51 (byte 6 upper nibble)
- 12 bits: rand_a (byte 6 lower nibble + byte 7)
- 2 bits: variant (`10`) at bits 64-65 (byte 8 upper 2 bits)
- 62 bits: rand_b (byte 8 lower 6 bits + bytes 9-15)

## Tests

8 unit tests in `mod tests`:
- `test_v4_random_format` - Validates v4 UUID format, version, and variant
- `test_v4_from_hex` - Tests hex-to-UUID conversion for v4
- `test_v7_format` - Validates v7 UUID format, version, and variant
- `test_v7_with_specific_time` - Verifies timestamp encoding in v7
- `test_v7_with_time_and_hex` - Tests v7 with both timestamp and hex constant
- `test_v7_min` - Tests v7 with minimum random value (all zeros)
- `test_v7_max` - Tests v7 with maximum random value (all ones)
- `test_parse_hex` - Tests hex string parsing with various formats

## Build & Test

```bash
cargo build          # Compile
cargo test           # Run tests
cargo clippy         # Lint
cargo fmt            # Format code
```

---
> Source: [coproduto/uuider](https://github.com/coproduto/uuider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
