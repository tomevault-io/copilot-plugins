## x07-registry-web

> - `bash scripts/check_generated_spec_mirror.sh`

# Repository Guide

## Build and check

- `npm run check`
- `npm run build`
- `bash scripts/check_generated_spec_mirror.sh`

## Published schema workflow

- `static/spec/` is a published surface for `x07.io/spec/...`.
- When X07 adds or changes public schemas, update the schema files here and keep `static/spec/index.json` in sync.
- Prefer mirroring canonical schema contents from `x07/spec/`; do not hand-edit divergent copies.
- `scripts/check_generated_spec_mirror.sh` auto-discovers sibling `../x07`, `../x07-wasm-backend`, and `../x07-platform-contracts` checkouts before falling back to `_deps/`, so run it from a normal `x07lang/` workspace before commit.
- Registry-web changes for installer work should usually stay limited to static docs/schema surfaces unless the UI itself needs to expose the new data.

---
> Source: [x07lang/x07-registry-web](https://github.com/x07lang/x07-registry-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
