## umbra

> **[📍 Back to Map](../CLAUDE.md)**

# Umbra — Context for Claude

**[📍 Back to Map](../CLAUDE.md)**

## 1. Project Overview

**Umbra** is the dedicated privacy and anonymity subsystem ("Shadow Layer") for the AntiGravity ecosystem. It provides metadata resistance and identity obfuscation for all station projects via Tor integration. The most active component is **Arti**, a Rust-based implementation of The Onion Router.

## 2. Technical Stack

- **Primary Language:** Rust (Arti) and C (legacy Tor)
- **Target:** Apple Silicon M5 with PQC support
- **Migration path:** C-Tor (Phase 1, legacy) → Arti/Rust (Phase 2, active) → Full Rust (Phase 3)

## 3. Architecture

```
umbra/
  arti/          — Rust-based Tor implementation (ACTIVE — primary focus)
  tor/           — C-based legacy Tor (Phase 1, maintenance only)
  MissionControl — Observability dashboard
  Guardian       — Network shield layer
```

**Third-party exclusion:** Do NOT modify code inside `umbra/tor/` (C-Tor) or `umbra/arti/` upstream source. Only modify AntiGravity integration layers.

## 4. Current Focus

Active development is on the **Arti** integration — Rust-based Tor for the station's privacy layer. Refer to `SYNC_LOG.md` for current phase status.

## 5. Workflows & Agent Expectations

- **Opening ritual:** `git pull` → read `SYNC_LOG.md`
- **TASKS.md contract:** Mark in-progress with `/`, complete with full checkmark, update `SYNC_LOG.md` after each task
- **Build:** `cargo build --release` with `-C target-cpu=native` for M5 optimization
- **Testing:** `cargo test` for all integration layers; do not write tests for third-party upstream code
- **License:** Apache-2.0 (cryptography project)
- **Tier:** Rigorous (PQC components, privacy-critical)

---
> Source: [roydsouza/umbra](https://github.com/roydsouza/umbra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
