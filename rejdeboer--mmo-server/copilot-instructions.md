## mmo-server

> This is a modern MMORPG monorepo using Rust and Bevy.

# Bevy MMORPG Project

This is a modern MMORPG monorepo using Rust and Bevy.

## Core Technologies

- Bevy: 0.18 (CRITICAL: Do not use Bevy 0.17 or older APIs. Ensure modern State and Plugin syntax is used)
- Axum web server
- Bitcode serialization
- Avian3d physics
- Deployed on Kubernetes

## Project Structure

The `crates/` directory contains the following crates:

- `game-server/` - A Bevy MMO game server
- `game-client/` - The Bevy client used by players to play the game
- `game-core/` - Rust code shared by the `game-client` and `game-server`
- `protocol/` - Bitcode structs used for communication between servers and clients
- `web-server/` - Axum server for auth, account / character management and in-game social features like guild / party chat and invites implemented using WebSockets
- `web-client/` - Client that talks to the web-server

## Networking & Protocol
- **No Serde for Game Loop:** Use `bitcode` for all `protocol/` structs sent between `game-client` and `game-server` to save bandwidth.
- **Shared State:** If a Component needs to exist on both the client and server, define it in `game-core`.
- When updating the `protocol`, ensure you update the serialization/deserialization on *both* the `game-server` and `game-client`.

## Rust Best Practices
- Keep imports clean. Group Bevy imports together (`use bevy::prelude::*;`).
- Use `clippy` to ensure idiomatic Rust. Handle all `Result` types properly; do not use `.unwrap()` in production server code unless absolute certainty exists.

## Deployment

This project is deployed using another repo containing Kubernetes manifests, Flux CD, Proxmox. This can be found at `../mmo-deployment`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rejdeboer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rejdeboer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
