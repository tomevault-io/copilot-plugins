## firmware

> MiSTer FPGA firmware replacement written in Rust, targeting the DE10-Nano (Intel Cyclone V SoC). Drop-in replacement for the MiSTer firmware with a modern codebase and JavaScript scripting support.

# 1FPGA Firmware

MiSTer FPGA firmware replacement written in Rust, targeting the DE10-Nano (Intel Cyclone V SoC). Drop-in replacement for the MiSTer firmware with a modern codebase and JavaScript scripting support.

## Project Structure

Monorepo with a Rust workspace (13 crates) and NPM workspaces (5 packages).

### Rust Crates (`src/`)

| Crate | Purpose |
|-------|---------|
| `firmware` | Main binary (`one_fpga_bin`). CLI entry point, initializes tracing, pins CPU core, launches script engine. |
| `one-fpga` | Core `Core` trait — generic interface for all emulator cores (ROM, save states, input, settings). |
| `mister-fpga` | MiSTer core implementation. SPI protocol, framebuffer, OSD, config strings, user I/O. |
| `cyclone-v` | Low-level Cyclone V FPGA register access via `/dev/mem` mmap. |
| `de10-nano` | DE10-Nano board support (I2C battery monitoring). |
| `firmware-ui` | UI application: menus, panels, OSD rendering, platform abstraction (DE10/desktop). |
| `firmware-gui` | Linux framebuffer GUI framework using `calloop` event loop. |
| `firmware-script` | Boa JavaScript engine integration. Exposes 1FPGA APIs to scripts. |
| `mister-fpga-ini` | MiSTer INI/JSON5 config file parser. |
| `fce-movie-format` | NES movie format parser. |
| `taser` | Movie player CLI utility. |
| `video-test` | Framebuffer rendering test (`/dev/fb0`). |
| `games-db-converter` | Game database CSV/XML/JSON converter. |

### JavaScript/TypeScript (`js/`)

| Package | Purpose |
|---------|---------|
| `js/frontend` | Main TypeScript frontend, compiled via Rollup to `dist/main.js`. Runs inside the Boa JS engine on device. |
| `js/frontend-react` | Next.js React frontend for development/debugging without hardware. |
| `js/1fpga/schemas` | Zod schemas for catalogs, settings. Generates JSON schemas and TypeScript types. |
| `js/1fpga/types` | TypeScript type definitions for `1fpga:*` module namespace. |
| `scripts/patreon` | Patreon API integration for patron credits. |

## Build

### Prerequisites
- Rust 1.88 stable (via `rust-toolchain.toml`)
- Node.js + npm
- Docker (for ARM cross-compilation)

### Key Commands

```bash
# Full build (frontend + ARM binary via Docker)
make build

# Frontend only
make build-frontend
# or: npm run build

# ARM binary only (requires Docker)
make build-1fpga

# Build and sign binary
make build-and-sign PUBLIC_KEY_PATH=/path/to/key

# Desktop build (no FPGA support, for UI development)
cargo run --bin one_fpga

# Run tests
cargo test          # Rust tests
npm test            # JS tests (@1fpga/schemas + @1fpga/frontend)

# Deploy frontend to device
make deploy-frontend  # rsync to MISTER_IP (default 192.168.1.79)

# Create new DB migration
make new-migration name=my_migration
```

### Docker Cross-Compilation

The ARM build uses a multi-stage Docker image (`docker/armv7/de10nano.Dockerfile`):
1. `cargo-chef` caches dependencies for fast rebuilds
2. Cross-compiles to `armv7-unknown-linux-gnueabihf`
3. Uses `mold` linker for speed

Target binary: `target/armv7-unknown-linux-gnueabihf/release/one_fpga`

### Feature Flags

- `platform_de10` — DE10-Nano hardware (default for ARM builds, `--no-default-features --features=platform_de10`)
- `platform_desktop` — Desktop simulator (default for `cargo run`)

## Architecture

### Boot Flow
1. `main.rs`: Parse CLI args, pin to CPU core 1, init tracing
2. `firmware_script::run()`: Initialize Boa JS engine
3. JS frontend (`main.js`) takes over: loads cores, manages UI

### Hardware Interaction
- **FPGA programming**: `cyclone-v` crate maps physical memory via `/dev/mem`
- **Core communication**: SPI protocol over 16-bit data bus (`mister-fpga`)
- **Display**: Linux framebuffer (`/dev/fb0`) + OSD overlay via SPI
- **Input**: SDL3 events → PS/2 scancodes / gamepad data → SPI commands

### Key Patterns
- `Core` trait in `one-fpga` — all emulator cores implement this
- `Rc<UnsafeCell<T>>` for single-threaded interior mutability (no Send/Sync)
- Unsafe code concentrated in: memory mapping, volatile register access, framebuffer lifetime management
- `thiserror` for error types throughout

## Code Style

### Rust
- Edition 2024 (some legacy crates still on 2021)
- `#[warn(unused_crate_dependencies)]` workspace-wide
- Release profile: LTO, panic=abort, stripped

### TypeScript/JavaScript
- Prettier: 100 char width, 2-space indent, single quotes, trailing commas
- Import ordering: `1fpga:*` first, then other namespaced, then `@/`, then relative
- Rollup for bundling with custom plugins (codegen, migrations, template literals)

## Testing
- Rust: `cargo test` — uses `rstest`, `pretty_assertions`
- JS: Jest with `ts-jest` — schema validation tests, version comparison tests
- Note: Rust tests require `cmake` installed locally (for native builds)

## Deployment

Target device: DE10-Nano at `MISTER_IP` (default `192.168.1.79`).

```bash
ssh root@$MISTER_IP 'killall MiSTer one_fpga'
scp target/armv7-unknown-linux-gnueabihf/release/one_fpga root@$MISTER_IP:/media/fat/one_fpga
ssh root@$MISTER_IP 'sync; /media/fat/one_fpga'
```

Frontend deployed via `make deploy-frontend` (rsync to `/root/frontend`).

## License

Apache 2.0

---
> Source: [one-retro/firmware](https://github.com/one-retro/firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
