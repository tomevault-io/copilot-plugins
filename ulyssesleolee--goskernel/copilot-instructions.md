## goskernel

> - This repository targets GOS v0.2: `bootstrap -> loader -> registry -> runtime`.

# GOS Project Rules (Cursor / AI Tooling Context)

## Architecture Baseline

- This repository targets GOS v0.2: `bootstrap -> loader -> registry -> runtime`.
- Treat `doc/RULE_GRAPH_PRIME.md` and `doc/GOS_GOVERNANCE_v0_2.md` as authoritative.
- `hypervisor/main.rs` is not a feature orchestration layer. It is bootstrap only.

## Development Constraints

- New plugins must be manifest-native and executor-driven.
- Do not add new `NodeCell`, `PluginEntry`, or `try_mount_cell` based plugins unless the plugin is explicitly approved as part of the legacy migration island.
- Every new `NodeSpec` and `EdgeSpec` must explicitly set `vector_ref`.
- Stable identities must use `derive_node_id` / `derive_edge_id`.
- Cross-plugin behavior must be expressed through manifests, capabilities, runtime edges, or runtime signal routing.

## AI Control Constraints

- AI nodes may orchestrate the graph, but must not bypass runtime permission checks.
- Do not introduce hidden direct-memory control paths for AI logic.
- AI decisions should flow through runtime-managed activation, scheduling, or control-plane interfaces.

## Required Verification

- Run `pwsh -File tools/verify-graph-architecture.ps1` before launch or merge.
- Run `cargo check -p gos-kernel` after structural changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UlyssesLeoLee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
