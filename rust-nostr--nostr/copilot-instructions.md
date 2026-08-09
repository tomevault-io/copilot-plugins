## nostr

> - Follow [CONTRIBUTING.md](CONTRIBUTING.md), including the organization guidelines

# Agent instructions

## Project conventions

- Follow [CONTRIBUTING.md](CONTRIBUTING.md), including the organization guidelines
  linked there.
- Follow the style and patterns of the crate and module being changed.
- Use the repository toolchain and formatting configuration.
- Write clean, self-documenting code with descriptive names, straightforward control
  flow, and small, focused units.
- Use comments to explain intent, invariants, or non-obvious tradeoffs, not to restate
  what the code already expresses.
- Keep changes focused. Do not mix unrelated refactors or formatting changes into
  the requested work.
- Prefer existing types and helpers over new abstractions or dependencies.

## Public API

- Keep the public API surface as small as possible. Use the narrowest visibility
  that satisfies the current requirement.
- Add public types, traits, methods, features, or re-exports only for a concrete
  user need.
- Preserve backward compatibility unless a breaking change is explicitly required.
- Document public API changes and update the affected crate's changelog when
  appropriate.
- Respect the workspace MSRV, feature boundaries, and `no_std` support where
  applicable.

## Performance

- Code must remain efficient on low-end and resource-constrained devices.
- Keep CPU, memory, allocation, storage, and network costs proportional to the work
  performed.
- Avoid unnecessary cloning, repeated parsing or serialization, redundant I/O, and
  unbounded collections, queues, tasks, or retries.
- Benchmark performance-sensitive changes when practical, and investigate meaningful
  regressions.
- Prefer clear, efficient code over speculative micro-optimizations. Explain
  non-obvious optimizations and their tradeoffs.

## Verification

- Add or update focused tests for behavior changes and important edge cases.
- Test the relevant crate, features, and targets for the change.
- Run `just fmt` while iterating and `just precommit` before considering the work
  complete.

---
> Source: [rust-nostr/nostr](https://github.com/rust-nostr/nostr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
