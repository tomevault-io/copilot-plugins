## daisy-rsx

> - For static-site crates and any `ssg_whiz` site builds, always run the binary from the site crate directory, not the workspace root. `ssg_whiz` currently resolves `dist/`,

  - For static-site crates and any `ssg_whiz` site builds, always run the binary from the site crate directory, not the workspace root. `ssg_whiz` currently resolves `dist/`,
  `assets/`, and `content/` relative to the process working directory. Example: use `workdir=/workspace/crates/example-site` with `cargo run --bin example-site` or `cd /
  workspace/crates/example-site && cargo run --bin example-site`. Do not use `cargo run -p <site>` from `/workspace` unless the site is updated to use absolute crate-root paths.

---
> Source: [bionic-gpt/daisy-rsx](https://github.com/bionic-gpt/daisy-rsx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
