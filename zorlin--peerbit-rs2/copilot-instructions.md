## peerbit-rs2

> * Strict, idiomatic Rust code

* Strict, idiomatic Rust code
* TEST DRIVEN DEVELOPMENT
* Do not simplify! Opt for improving and fixing a broken feature rather than ripping it out
* Do not make the user do the testing - that's what test suites are FOR
* Use Playwright and DeepWiki MCPs extensively

## Testing Rules

**ALL PeerBit testing MUST be done via cargo test:**
```bash
cargo test -p peerbit-server test_name -- --ignored --nocapture
```

Example:
```bash
cargo test -p peerbit-server test_raw_libp2p_connection -- --ignored --nocapture
```

**DO NOT:**
- Start the relay server manually with `./target/release/peerbit-server`
- Run standalone processes for testing
- Use `run.sh` or similar scripts for testing

## PeerBit Protocol Architecture

**CRITICAL**: PeerBit does NOT use standard libp2p GossipSub! It uses custom protocols:

### Custom Protocols (NOT standard libp2p)
1. **`/lazysub/0.0.0`** - DirectSub pubsub protocol (custom, NOT GossipSub!)
   - Uses length-prefixed binary encoding, NOT CBOR
   - Implementation: `crates/peerbit-transport/src/lazysub_behaviour.rs`
   - Exported as: `LazySubBehaviour`, `LazySubCodec`, `LazySubEvent`

2. **`/peerbit/direct-block/0.0.0`** - DirectBlock for distributed block storage
   - Custom block transfer protocol
   - Implementation: `crates/peerbit-transport/src/direct_block.rs`

3. **`/peerbit/direct-sub/0.0.0`** - Alternative DirectSub protocol ID
   - Part of the DirectSub system
   - Implementation: `crates/peerbit-transport/src/direct_sub.rs`

### Standard libp2p Protocols (relay server requirements)
- **Circuit Relay V2** (`/libp2p/circuit/relay/0.2.0`) - For browser relay
- **WebSockets** (`/ws`, `/wss`) - Browser connectivity
- **WebRTC** (`/webrtc/v1`) - Browser-to-browser direct connections
- **Identify** - Peer identification
- **Ping** - Connectivity testing

### DO NOT:
- Add standard GossipSub - PeerBit doesn't use it
- Use CBOR encoding for `/lazysub/0.0.0` - it has its own codec
- Assume standard libp2p pubsub patterns - DirectSub is custom

### References:
- DeepWiki: dao-xyz/peerbit repository
- Transport layer: `crates/peerbit-transport/`
- Server implementation: `crates/peerbit-server/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zorlin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Zorlin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
