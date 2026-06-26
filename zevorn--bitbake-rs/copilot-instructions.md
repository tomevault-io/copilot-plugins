## bitbake-rs

> These rules apply to the whole `bitbake-rs` repository.

# AGENTS.md

# Scope
These rules apply to the whole `bitbake-rs` repository.

# Required reading
Before changing code, tests, or documentation, read the project style guide:

- `docs/style/bitbake-rs-style-guide.md`

Then read the relevant upstream reference for the area you touch:

- BitBake behavior, Python runtime compatibility, metadata syntax, manuals, and developer workflow:
  - `docs/upstream/bitbake/`
  - `docs/upstream/yocto-contributor-guide/`
- Rust implementation style:
  - `docs/upstream/asterinas-coding-guidelines/`
  - `docs/upstream/asterinas-coding-guidelines/rustfmt.toml`
- Source snapshot provenance:
  - `docs/upstream/README.md`

# Rules
- Keep upstream documentation under `docs/upstream/` as imported reference material; do not rewrite it for `bitbake-rs` architecture.
- Put project-specific style or architecture guidance outside `docs/upstream/`.
- Match BitBake/OpenEmbedded commit style: `<component>: <imperative summary>`.
- Sign commits with `Signed-off-by: Name <email>` and do not add AI attribution trailers.
- Preserve BitBake names and semantics for Python-facing APIs and compatibility behavior.
- Use the 星绽 OS/Asterinas guidance as the baseline for Rust clarity, safety, visibility, documentation, and allocation-conscious code.

---
> Source: [zevorn/bitbake-rs](https://github.com/zevorn/bitbake-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
