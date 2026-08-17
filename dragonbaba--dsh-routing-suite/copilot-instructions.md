## dsh-routing-suite

> - Keep the product a lightweight, ready-to-use standard DSH bundle; do not add preset-copy CLIs or install lifecycle mutations.

# Project Instructions

- Keep the product a lightweight, ready-to-use standard DSH bundle; do not add preset-copy CLIs or install lifecycle mutations.
- Host runtime may only add non-destructive routing guidance and a read-only status endpoint. Do not add filesystem/process/package management, dynamic code execution, tool filtering, or extra LLM calls.
- Preserve existing prompt sections, contexts, tools, and selected preset behavior.
- Keep the UI read-only and localized from the primary machine locale: Simplified Chinese for `zh-CN`/`zh-Hans`, English otherwise.
- Keep Host, Client, patch, and package identities aligned through contract and artifact tests.
- Runtime and package code are MIT. Preserve reviewed-project attribution in provenance/acknowledgements without copying removed upstream implementations.
- Run `npm run verify`, inspect the exact dry-run tarball, and perform a temporary-profile DSH install before release.
- Do not commit, push, tag, or publish without explicit user approval for the current task.

---
> Source: [dragonbaba/dsh-routing-suite](https://github.com/dragonbaba/dsh-routing-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
