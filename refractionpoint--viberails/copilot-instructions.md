## viberails

> After cloning, install the git hooks:

# Project West Coast

## Setup

After cloning, install the git hooks:

```bash
curl -fsSL https://get.viberails.io/install.sh | bash
```

## Development Requirements

- Run `cargo clippy -- -D warnings` before committing (no warnings allowed)
- Run `cargo test` before committing (all tests must pass)

These checks are enforced by the pre-commit hook.

## Build & Test Commands

- Build: `cargo build`
- Test: `cargo test`
- Clippy: `cargo clippy -- -D warnings`

## Providers

Whenver you have to make changes for providers, providers are implementing
the LLmProviderTrait trait and they can override the default implemtation
this way we don't break other working providers

---
> Source: [refractionPOINT/viberails](https://github.com/refractionPOINT/viberails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
