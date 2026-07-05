## nightly-rustfmt

> Enforce nightly rustfmt style for Rust code generation


# Nightly Rustfmt Requirement

When generating or editing Rust code in this repository:

- Produce code that matches nightly rustfmt output, not stable-only formatting.
- Follow the repository `rustfmt.toml` style, including import grouping and line wrapping behavior expected by nightly rustfmt.
- Avoid committing formatting that would be rewritten by pre-push hooks.
- If Rust files are changed, run `cargo +nightly fmt` before finalizing changes whenever possible.
- Treat nightly rustfmt compliance as required for completion.

## Quick Examples

- Prefer grouped crate imports when nightly rustfmt would merge them.
- Wrap long doc comments and long string literals as nightly rustfmt expects.

---
> Source: [AlephantAI/AIephant-AI-Agent-Gateway](https://github.com/AlephantAI/AIephant-AI-Agent-Gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
