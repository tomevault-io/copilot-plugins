## crabllm

> See [CONTRIBUTING.md](CONTRIBUTING.md) for codebase architecture.

See [CONTRIBUTING.md](CONTRIBUTING.md) for codebase architecture.

## Provider crate has external consumers

The provider crate is a public library — other products depend on it directly,
not just the proxy. Never remove methods from the `Provider` trait (e.g.
`anthropic_messages_stream`) just because the proxy doesn't call them.

---
> Source: [crabtalk/crabllm](https://github.com/crabtalk/crabllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
