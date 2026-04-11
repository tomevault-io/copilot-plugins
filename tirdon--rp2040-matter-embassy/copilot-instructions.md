## rp2040-matter-embassy

> A Matter-compatible On/Off Light implementation for the Raspberry Pi Pico W (RP2040), written in Rust using the Embassy async runtime and the `rs-matter-embassy` stack.

# Matter Light (Raspberry Pi Pico W) — Project Context

A Matter-compatible On/Off Light implementation for the Raspberry Pi Pico W (RP2040), written in Rust using the Embassy async runtime and the `rs-matter-embassy` stack.

## Technical Architecture

- **Hardware:** Raspberry Pi Pico W (RP2040 + Infineon CYW43439 WiFi/BLE chip).
- **Runtime:** [Embassy](https://embassy.dev) async framework for embedded Rust.
- **Protocol:** [Matter](https://csa-iot.org/all-solutions/matter/) implementation via [rs-matter](https://github.com/sysgrok/rs-matter) and [rs-matter-embassy](https://github.com/sysgrok/rs-matter-embassy).
- **Transport:** WiFi for main communication, BLE for commissioning.
- **Memory Management:** `no_std` with `embedded-alloc` for small dynamic allocations (X.509 cert parsing) and `static_cell` for long-lived stack components.

## Key Files

- `src/main.rs`: Core logic, Matter stack initialization, endpoint/cluster definitions.
- `Cargo.toml`: Dependency configuration (uses `patch.crates-io` for `rs-matter` dev branches).
- `build.rs`: Linker configuration and automatic CYW43 firmware download.
- `memory.x`: RP2040 memory layout (2MB Flash, 264KB RAM).
- `.cargo/config.toml`: Target (`thumbv6m-none-eabi`) and runner (`probe-rs`) settings.

## Development Workflow

### Prerequisites

```sh
rustup default nightly
rustup target add thumbv6m-none-eabi
cargo install probe-rs-tools
```

### Common Commands

- **Build:** `cargo build` (downloads required CYW43 firmware on first run).
- **Flash & Run:** `cargo run --release` (requires a debug probe like a Picoprobe or CMSIS-DAP).
- **Logging:** Uses `defmt` over RTT. `probe-rs run` will capture and display logs.

### Adding New Clusters

Follow the pattern in `src/main.rs`:
1. Import cluster handler from `rs_matter_embassy::matter::dm::clusters`.
2. Instantiate the cluster handler (e.g., `on_off::OnOffHandler`).
3. Chain it to the `handler` logic using `EpClMatcher`.
4. Update the `NODE` static definition to include the new cluster in the appropriate endpoint.

## Development Conventions

- **Async First:** All IO and stack operations are async via Embassy.
- **Static Allocation:** Prefer `static_cell` or `mk_static!` macro for long-lived objects to avoid heap fragmentation.
- **Logging:** Use `defmt::info!`, `defmt::debug!`, etc. Avoid `println!` or `std` macros.
- **Error Handling:** Use `unwrap!` from `defmt` or explicit error matching; panics are captured by `panic-rtt-target`.

## Configuration Constants

- `BUMP_SIZE`: Memory reserved for the Matter stack bump allocator (approx 16.5KB).
- `LIGHT_ENDPOINT_ID`: The ID for the main light endpoint (usually 1).
- `NODE`: The root definition of endpoints and clusters.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tirdon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tirdon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
