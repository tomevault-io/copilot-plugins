## rust-solid-hexagonal

> Rust — SOLID and ports-and-adapters (traits in core, I/O at edges)


# Rust — SOLID and hexagonal layout

- **S** — One `struct`/`impl` per reason to change. Separate use-case orchestration from serialization or SQL details.
- **O** — Extend with new types or **trait** implementations instead of growing a central `match` on strings; prefer enums when the set is closed and known.
- **L** — Types that implement domain traits must honor the contract (no unexpected panics in methods the core treats as total).
- **I** — Small, client-oriented traits (`SessionRepository`, `UplinkRepository`), not a single persistence “god trait”.
- **D** — **Core/domain** depends on abstractions (`trait` in `ports`); **adapters** (sqlite, udp, …) implement those traits and depend on domain types, not the reverse.
- **Hexagonal**: business rules without `tokio::net`, `rusqlite`, or filesystem paths in the nucleus—only types and ports. Side effects live at the edges.

---
> Source: [antonygiomarxdev/maverick](https://github.com/antonygiomarxdev/maverick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
