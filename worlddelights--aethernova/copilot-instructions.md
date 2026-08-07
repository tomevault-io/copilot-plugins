## aethernova

> Project AetherNova - Another name considered aethnervola.

# Overview

Project AetherNova - Another name considered aethnervola.

Here is a comprehensive set of specifications for an exploration-focused, WebGPU-based peer-to-peer voxel engine designed to support multiple, physically distinct universes.

## Core Vision

A massively scalable, in-browser voxel engine utilizing WebGPU for compute-heavy raymarching and physics. The platform is designed as a federated, decentralized network of "universes," where players explore and morph their avatars rather than engage in combat or competition. The architecture relies on WebRTC/WebTransport for peer-to-peer (P2P) networking, ensuring the server acts merely as a signaling relay, not a physics authority.

## Technical Stack

* **Rendering & Compute:** WebGPU (WGSL Compute Shaders, Raymarching).
* **Local Processing:** WebAssembly (Wasm) compiled via Rust/C++, utilizing SIMD and SharedArrayBuffers for terrain generation.
* **Networking:** WebRTC DataChannels (primary P2P transport) or WebTransport (for reliable/unreliable stream multiplexing if browser support permits).
* **Local Storage:** Origin Private File System (OPFS) for fast, synchronous chunk loading/saving.
* **Signaling Server:** Lightweight Node.js/Go server for P2P connection brokering (STUN/TURN handling).

---

## 1. Engine & Rendering Architecture

* **Structure:** Sparse Voxel Directed Acyclic Graphs (SVDAGs) stored in WebGPU Storage Buffers, allowing massive scale by deduplicating identical regions of space.
* **Rendering:** 100% Compute Shader Raymarching. No triangle rasterization. Uses a low-resolution brickmap pre-pass for hierarchical frustum culling to ensure only relevant 3D space is evaluated.
* **Scale:** Variable voxel sizing based on LOD depth, allowing transitions from massive macro-structures down to granular micro-details.

---

## 2. Construction & Automata Mechanics

To enable the creation of visually stunning, self-contained mechanisms, the engine supports voxel-level programmable logic.

* **3D Cellular Automata:** Voxels can act as logic gates or cellular machines. A compute pass evaluates a voxel's state based on its immediate Moore neighborhood (the surrounding 26 voxels). This allows players to build massive, self-propelling engines, rhythmic light structures, or automated sorting systems entirely out of static blocks.
* **SDF Catalysts (The Player Tool):** The player Avatar acts as a volumetric catalyst (Defined by a Signed Distance Field). Instead of placing blocks one by one, the player projects localized mathematical rules (e.g., "crystallize," "decay," "attune") that interact with the environment, sculpting structures dynamically.

## 3. Explored Physics Prototypes (To Be Tested)

These concepts represent the initial suite of modular rules to test in the prototype sandbox:

* **Dimensional Bitmasking (Visibility & Collision):** Voxels and players share a `strata_mask`. Collision and rendering evaluate `(voxel.mask & player.mask) != 0` as a hyper-fast $O(1)$ bitwise check. Allows overlapping, non-interfering structures and realities in the same physical space.
* **Adhesion & Decay Kinetics:** Replacing gravity with surface tension. Matter drifts linearly until it contacts a structure, calculating an attachment bond ($A_{bond}$) based on surface area and material type. Bonds can decay over time or via heat, causing massive structures to dynamically calve, fracture, and drift apart.
* **State Simplification:** Elimination of continuous gaseous states. The universe relies strictly on Solids (permanent adhesion) and Liquids (instant adhesion with rapid decay, causing material to "crawl" over surfaces).

## 4. Decentralized Multiplayer Architecture

* **Signaling Relay:** A lightweight server (Node/Go) that handles spatial brokering (matching players in proximity) and WebRTC/WebTransport STUN/TURN handshakes. It holds no physics authority.
* **P2P State Authority:** The first player to load a macro-region acts as the temporary host, processing the Wasm generation and syncing the initial SVDAG.
* **Delta Syncing:** Peers transmit only state changes (e.g., "Voxel at $X,Y,Z$ changed from state A to B") via WebRTC DataChannels, keeping bandwidth minimal.

## 5. Avatar & Player Mechanics

The player is not a fixed humanoid model. The core entity is a "Consciousness Point" (an energy spark), which can dynamically manipulate its physical manifestation based on the environment.

### The Morphing System

* **The Spark:** The baseline avatar. Capable of high-speed, frictionless movement through low-density mediums.
* **Resonance Forms:** By interacting with specific voxel types, the player can absorb their properties to build a temporary physical shell.
* *Example:* Absorbing "Heavy Silicate" allows the avatar to sink to the bottom of liquid methane oceans and interact with high-pressure switches.
* *Example:* Absorbing "Aero-Gel" expands the avatar's volume, allowing them to ride thermal updrafts or float in low gravity.

* **Volumetric Avatars:** Avatars are constructed from voxels dynamically. When morphing, compute shaders generate the new avatar shape via signed distance fields (SDFs) transitioning between states.

## 4. Networking: Decentralized Multiplayer

The goal is a serverless exploration experience once the connection is made.

### The Signaling Relay (Server-Side)

* **Proximity Brokering:** The server tracks rough spatial coordinates of players. When Player A and Player B enter the same region (e.g., the same macro-chunk), the server initiates a WebRTC/WebTransport handshake.
* **No State Authority:** The server does not calculate physics, player health, or inventory. It only relays IPs and cryptographic keys for the P2P connection.

### Peer-to-Peer Data Synchronization (Client-Side)

* **State Authority:** The player who first enters an unloaded region becomes the temporary "Host" for that chunk. They generate the terrain via their local Wasm worker and sync the SVDAG data to peers.
* **Delta Compression:** Instead of sending entire chunk states, peers only transmit voxel modifications (e.g., "Voxel at $X,Y,Z$ changed frequency from A to B").
* **WebRTC vs WebTransport:** WebRTC DataChannels provide battle-tested, low-latency peer-to-peer connectivity. However, keeping an eye on WebTransport is crucial, as it offers a cleaner QUIC-based pipe, though browser support for direct P2P WebTransport (bypassing a server) remains an evolving area.

## Critical Agent Goal: Pluggable Modularity

To find the right balance for structure-building and automata, the engine must not hardcode physics. The core architecture must act strictly as a renderer and state-manager, while the actual rules of the universe are modular plugins.

* **WGSL Shader Injection:** Physics rules (movement, adhesion, decay, state changes) are written as isolated WGSL functions. The JavaScript/Wasm host dynamically strings these together and recompiles the compute pipeline when testing new mechanics or transitioning between universes.
* **Decoupled State vs. Rule:** The SVDAG stores raw voxel state (e.g., a 16-bit payload). What that payload *means* (temperature, pressure, bond strength, or automata logic state) is entirely dependent on the currently injected physics module.
* **Wasm Rule Evaluators:** For complex, large-scale generation or non-GPU friendly logic, Wasm modules (written in Rust/C++) evaluate the rules and push deltas back to the GPU via `SharedArrayBuffer`.

---
> Source: [worlddelights/aethernova](https://github.com/worlddelights/aethernova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
