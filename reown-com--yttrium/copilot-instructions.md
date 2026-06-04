## yttrium

> Rust `just lint` as your linting tool. It will internally run `cargo fmt` and `cargo clippy` with the full configuration.


## Workflow

Rust `just lint` as your linting tool. It will internally run `cargo fmt` and `cargo clippy` with the full configuration.

## Rust coding rules

- Avoid creating functions unless they are necessary to reduce a significant amount of code duplication
- Avoid extra variable bindings
- Make the code and tests consise
  - Avoid adding comments and newlines unless they are important for code readability

## Playwright rules

- Avoid `waitForTimeout()` at almost all costs (the test itself should not be dependant on timing). There must be a very good reason to add a timeout.

---
> Source: [reown-com/yttrium](https://github.com/reown-com/yttrium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
