## rboot

> Always run the following checks and ensure they pass:

# Development Guidelines

## Before committing

Always run the following checks and ensure they pass:

```sh
cargo fmt --check
cargo clippy --target x86_64-unknown-uefi
```

## Build

```sh
cargo build --target x86_64-unknown-uefi
```

---
> Source: [rcore-os/rboot](https://github.com/rcore-os/rboot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
