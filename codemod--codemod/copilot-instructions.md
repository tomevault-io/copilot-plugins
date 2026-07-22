## codemod

> This crate implements JavaScript/TypeScript semantic analysis on top of OXC and resolver behavior.

# JavaScript Semantic Provider Rules

This crate implements JavaScript/TypeScript semantic analysis on top of OXC and resolver behavior.

- Keep lightweight and accurate provider paths behaviorally aligned where their contracts overlap.
- Preserve Yarn PnP behavior behind the existing feature flag.
- Add fixtures for resolver, module graph, and reference edge cases when changing lookup behavior.
- Validate with `cargo test -p language-javascript` and sandbox semantic integration tests when
  provider behavior changes.

---
> Source: [codemod/codemod](https://github.com/codemod/codemod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
