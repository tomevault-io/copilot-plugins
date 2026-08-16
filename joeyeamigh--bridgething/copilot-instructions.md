## bridgething

> Rules an agent has to follow when editing this repo. The codebase has a

# bridgething agent guide

Rules an agent has to follow when editing this repo. The codebase has a
clean lib/core split that is easy to silently break - these rules exist
to keep that split intact.

## Crate layout and what goes where

The repo splits into two top-level workspace roots:

- `crates/` — Rust workspace members. Two families: the daemon side (`crates/lib`, `crates/core`, `crates/iap2`, `crates/mfi`, `crates/mfi-proxy`, `crates/dsp`, `crates/wakeword`, ...) and the shared companion core (`crates/sdk-runtime`, `crates/io`, `crates/gateway-rs`, `crates/delivery/{core,napi,wasm}`, `crates/companion`, plus `crates/spotify` and `crates/nlu` linked into it). `crates/client-rs` and `crates/host-gateway` are the Rust-side consumers. The cargo workspace also pulls in `tools/codegen/` and `desktop/src-tauri/`.
- `packages/` — Bun/turbo workspace members (`packages/browser`, `packages/client-ts`, `packages/ui`, `packages/updater`, `packages/session-rn`, `packages/webapp-shared`, `packages/webapps/{builtin,catalog}/*`). `builtin` rides the daemon release and is never published to the catalog; `catalog` is what the store distributes. `packages/ui` is preact + tailwind shared by the desktop app and the site. `packages/companion/{swift,kotlin}` and `packages/asr` are the mobile platform shells over the shared core, not bun members.
- `mobile/` — RN app, consumer of the packages.
- `desktop/` — tray-resident Tauri app, laid out the way `create-tauri-app` scaffolds one: the app root is `desktop/` itself (`package.json`, `index.html`, `vite.config.ts`, preact sources under `src/`) with `src-tauri/` beside them. The two deviations from the scaffold are deliberate: `src-tauri/` is a member of the root cargo workspace, and the frontend is a bun workspace member (`@bridgething/desktop-frontend`) so it shares `packages/ui`. `src-tauri/` links `crates/companion` natively and is the only place state lives; the frontend holds none of it.
- `site/` — bridgething.com. Astro + preact islands, deployed to cloudflare.

The lib/core split below is the load-bearing one. Naming convention: Rust crates use kebab-case package names (`bridgething-mfi`); TS packages use scoped names (`@bridgething/lib`).

### crates/lib/ (`libbridgething`) — wire surface only

Anything that crosses a websocket or bluetooth boundary lives here, plus
the codec/framing for those boundaries. That's it.

Allowed in lib:

- Wire DTOs (every type that gets serialized to msgpack on the BT link
  or to JSON on the local websocket).
- The codec / framing in `crates/lib/src/protocol/`.
- Compile-time constants used by the protocol (UUIDs, ports, class IDs).
- `serde`, `ts-rs`, `uuid`, `serde_with`, `derive_more`,
  and the `protocol` feature deps (tokio-util, flate2, rmp-serde).

Forbidden in lib:

- Tokio runtime types (`tokio::sync::mpsc`, `tokio::task`, etc).
- Handlers, managers, daemon state, hardware drivers.
- Errors that aren't pure protocol errors. `EndecError` is fine.
  `BluetoothError` is not.
- Anything that would not be useful to a third party importing
  `libbridgething` purely to speak the wire protocol.

If you reach for `tokio::sync::mpsc::Sender<Foo>` in lib, stop — that
type belongs in core.

### crates/core/ (`bridgething`) — the daemon

Everything else. Handlers, managers, axum server, BlueR plumbing,
chromium CDP driver, persistent state, hardware drivers (ALS, mic),
systemd integration. The binary lives here.

Core depends on lib for wire types and re-exports nothing.

### crates/client-rs/, packages/browser/

Both consume `libbridgething`. They MUST NOT redefine wire types — they
re-export from lib or build on top of lib's types. If a wire type needs
a field added, the field goes in lib and propagates outward, never the
other way.

`packages/browser` is the host-side transport: the delivery core compiled
to wasm, plus the Web Serial and WebSocket plumbing that has to be written
in TypeScript because no other language can reach those APIs. The protocol
logic underneath it is Rust and is not duplicated there. A host reaches the
daemon's network gateway on port 8892 the same way on Linux, macOS, Windows
and in the browser.

## Wrap, don't duplicate

If a runtime variant of an enum doesn't fit a wire enum, do not copy
the wire enum into core and add the variant. Wrap it.

Example of the right pattern (already in `crates/core/src/handler/client/msg.rs`):

```rust
// lib::ClientCommandType is the wire enum.
// core::RecvMsgData wraps it and adds runtime-only variants.
pub enum RecvMsgData {
  Bluetooth(ClientBluetoothCommand),    // re-projected from lib variant
  // ...
  Hole,                                 // runtime only
  Unsupported(PossibleRecvMsg),         // runtime only
  ChangeMode(ClientMode),               // runtime only
  ConnectionClosed(u16, String),        // runtime only
}
```

The fields inside `Bluetooth(ClientBluetoothCommand)` are NOT redefined
in core — they reuse the lib type. Only the _enum shell_ is core-side
because it has runtime-only variants.

The cautionary tale: `WebappInfo` was at one point copy-pasted
identically between lib and core. There is exactly one canonical home
for any wire type: `crates/lib/`. Core imports it.

If you find yourself writing a `struct` or `enum` in core that has the
same fields (or near-identical fields) as one in lib, stop. The lib
type either:

- already does what you need — import it, or
- needs a runtime extension — write a wrapper that holds it.

## Stock translation lives in core, deliberately

The stock Spotify webapp's wire protocol (raw JSON the unmodifiable
stock webapp emits and consumes) lives in `crates/core/src/stock/` and the
dispatcher that translates it lives in `crates/core/src/handler/client/stock.rs`.
This is the one apparent exception to "wire types live in lib" and it
is intentional.

Why: the SDKs that consume `libbridgething` (gateway, mobile apps,
on-device clients) don't speak stock. Stock is a translation layer at
the daemon edge — modern shapes go over BT, stock JSON only ever lands
on a local websocket from the stock webapp. Putting stock in lib would
pollute every generated TS binding with types those consumers
will never use.

There IS a `crates/lib/src/stock/` module with a small handful of types
(`StockSetPreset`, `StockPreset`). Those are not the same thing — they
are SDK-facing types that a _modern_ webapp uses to invoke legacy
operations through `ClientCommandType::LegacyStock`. The rule:

- `crates/lib/src/stock/` = SDK-facing types for legacy operations a modern
  webapp may want to invoke. Generated TS gets these.
- `crates/core/src/stock/` = wire shapes for the stock Spotify webapp. Never
  leaves the daemon.

If you're adding a type because the stock webapp sends or receives it,
that goes in `crates/core/src/stock/`. If you're adding a type that a
modern webapp wants to access from its TypeScript code, that goes in
`crates/lib/src/stock/`.

## shared/ in lib

`crates/lib/src/shared/` is for types used by both directions of a wire
protocol (gateway↔bridge, client↔server) AND used by more than one
protocol or surface. Examples that belong: `Track`, `Album`, `Device`, `WebappInfo`.

If a type is only used in one direction or only by one protocol, it
goes in that direction's module. Don't promote to `shared/` for tidiness.

## Codegen — run `just codegen`, never hand-edit generated files

`just codegen` runs the `bridgething-codegen` tool over `crates/lib/src/`
and emits three arms:

- **ts**: `crates/lib/ts/bindings/` (wire DTOs for the webapp client) plus
  `crates/companion/ts/companion.ts` via ts-rs, then prettier. There is no
  generated TS gateway dispatch: the browser speaks the wire through the Rust
  core in `packages/browser`, so a second implementation of it would be the
  thing this tool exists to prevent.
- **rust**: the `bridgething-client` / `bridgething-gateway` surface files
  (`crates/*-rs/src/surface.generated.rs`) — naming sugar over the generic
  runtime, no DTOs materialized.
- **docs**: `crates/lib/docs/surfaces.json`, the client-SDK reference IR
  the site renders.

There is no swift or kotlin schema arm: both mobile apps consume the shared
Rust core over uniffi, and those bindings are regenerated with
`just companion-bindings` after any change to the core's FFI surface
(`crates/companion`).

Generated files have no human edits. If a generated file is wrong, the
fix goes in:

1. The Rust source in `crates/lib/src/` (annotations, types).
2. The codegen tool in `tools/codegen/` (post-processing transforms).

Never add another perl/sed one-liner to the Justfile to patch generated
output. Every transform belongs in the codegen tool, where it is
discoverable, testable, and reviewed alongside the type that needs it.

## Concurrency + ownership

The daemon is actor-style: one tokio task owns its state, and every state
transition flows through messages, not locks. `RfcommGateway` in
`crates/core/src/bluetooth/rfcomm/mod.rs` is the canonical example to
copy when building a new subsystem.

Rules:

- **One task owns its data; other tasks reach it via mpsc.** In
  `RfcommGateway::recv`, the `connections: HashMap<Address, Connection>`
  is owned by the `recv()` task and only mutated inside its `select!`
  loop. Outside callers post messages through `GatewaySendTx`. Do not
  reach for `Arc<Mutex<Foo>>` to "share" Foo — either Foo's owner
  spawns a task and accepts commands by mpsc, or Foo is cheaply
  cloneable wire data.
- **`Arc<RwLock<...>>` exists in this codebase but is the exception.**
  `state::AppState` and `ProfileManager::profile_state` both wrap
  read-mostly snapshot state read by many surfaces (HTTP, BT, agent).
  New subsystems should reach for an mpsc before reaching for a lock;
  locks are tolerated when the data is read-mostly snapshot, not when
  it's the hot path of a protocol state machine.
- **`init() -> Self` and `spawn(self) -> JoinHandle<()>` are separate.**
  Construction takes ports, registers BlueZ profiles, opens sockets;
  `spawn` consumes the struct and returns a handle so the daemon can
  decide when loops start. See `RfcommGateway::init` /
  `RfcommGateway::spawn`. Background tasks return `JoinHandle<()>`
  stored in `_handle: JoinHandle<()>` fields — the underscore says
  "kept alive to drop on Drop, never `.await`ed".
- **Bounded mpsc, default capacity 16.** `tokio::sync::mpsc::channel(16)`
  appears in `BluetoothManager::init`, `RfcommGateway::init`,
  `GatewayCon::init`. The bound is the backpressure — propagate
  upstream rather than allocating an unbounded buffer.
- **Byte-stream protocols use `tokio_util::codec::Framed`.** The rfcomm
  gateway frames `BridgeEndec` over `bluer::rfcomm::Stream`; the iap2
  link layer frames `LinkCodec` over the same. Implement `Decoder` to
  return `Ok(None)` until enough bytes are buffered, and to advance one
  byte on resync paths. `Framed::split()` gives `Sink + Stream` halves
  you can move into separate tasks.
- **Heap discipline on protocol code.** Use `bytes::Bytes` for
  zero-copy shared-readonly buffers and `bytes::BytesMut` for
  build-phase. Avoid `Vec<u8>` clones for in-flight payloads:
  retransmit queues hold `Bytes` (refcount-only clone), CSM builders
  write into a single `BytesMut` then freeze.
- **Don't heap clone** unless absolutely necessary. Consider other
  alternatives first, or make sure the clones are gated by connected
  gateway/client reality and not fanned out unnecessarily.
- **Tracing density matches `RfcommGateway`'s.** Every state transition
  at `debug!`, every frame at `trace!`, every successful handshake /
  connection-up at `info!`, every error path at `warn!` or `error!`.
- **`thiserror` enums with `#[from]` for error conversion.** The shape
  of `BluetoothError` is the model: each subsystem gets its own enum,
  and the larger error types `#[from]` the smaller ones so `?` works
  through layers.

The Car Thing has 512 MB of RAM shared with chromium and the kiosk web
app. Heap clones, unbounded queues, and mutex contention all cost real
frames. The actor-with-bounded-mpsc shape is what keeps the hot paths
cheap.

## Workspace ergonomics

- `cargo run -p bridgething` runs the daemon against dev paths under
  `~/.local/share/bridgething/` and `~/.config/bridgething/`. See
  `crates/core/src/paths.rs` for env var overrides. It opens a bluez
  session and reconfigures the host adapter, so for host iteration use
  `just dev-daemon` (or `just dev-daemon-start`), which leaves the radio
  alone and keeps its state under `.dev/`.
- `cargo build -p bridgething --features superbird --no-default-features`
  is the on-device build (drops dev-host features). `just cross-build`
  runs those flags inside the aarch64 build image.
- `cargo test -p libbridgething` runs unit + golden tests. The golden
  fixtures live in `crates/lib/tests/`; regenerate with
  `UPDATE_GOLDEN=1 cargo test -p libbridgething --test golden`.
- `just test-all` runs every language suite; `just test-{rust,kotlin,swift,ts}`
  runs one. `FORCE=1` reaches the cache-backed gradle and turbo suites.
- `just codegen` after any change to a lib type that crosses to TS;
  `just companion-bindings` after any change to the shared core's
  FFI surface.
- No emdashes or endashes.
- NEVER #[allow(dead_code)]. Period. It's a useful metric.
- Comments in core/ should be MINIMAL and only for gotchas (of which
  there should not be many, because it usually means bad code).

---
> Source: [JoeyEamigh/bridgething](https://github.com/JoeyEamigh/bridgething) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
