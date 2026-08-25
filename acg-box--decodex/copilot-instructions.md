## decodex

> This repository uses `openwiki/` as its project knowledge entrypoint.

## OpenWiki

This repository uses `openwiki/` as its project knowledge entrypoint.

Start here:
- [OpenWiki quickstart](openwiki/quickstart.md)

OpenWiki includes repository overview, architecture notes, workflows, domain concepts, operations, integrations, testing guidance, and source maps.

When working in this repository, read the OpenWiki quickstart first, then follow its links to the relevant architecture, workflow, domain, operation, and testing notes.

## Rust toolchain

- Use the `stable` Rust compiler channel for every active Rust workspace.
- Keep each active `rust-toolchain.toml` channel set to `"stable"`.
- Do not replace `stable` with a numbered, beta, or nightly compiler channel unless the
  user explicitly authorizes that change.
- Build and test commands must not override this policy with a numbered toolchain.

---
> Source: [acg-box/decodex](https://github.com/acg-box/decodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
