## rust-style

> When any Rust code is modified we should always do some basic checks to ensure it meets our standards.


When any Rust code is modified we should always do some basic checks to ensure it meets our standards.

- run `cargo check` to ensure the code is buildable
- run `cargo fmt --all` to reformat the code
- run `cargo clippy -- -D warnings` and fix any issues that are reported
- run `cargo nextest r` to ensure unit tests still pass
- run `yarn build-native` to ensure that the Rust bindings for Atlaspack still build correctly

---
> Source: [atlassian-labs/atlaspack](https://github.com/atlassian-labs/atlaspack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
