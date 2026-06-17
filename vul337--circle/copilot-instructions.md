## circle

> This repository is the CiRCLE project. Changes should keep

# CiRCLE Repository Notes

## Scope

This repository is the CiRCLE project. Changes should keep
the tree runnable, readable, and safe to share.

## Repository Rules

- Keep the public example as a real directory at `example/`. Do not turn it
  into a symlink.
- Use `evaluation/example/` only for supplementary small cases such as
  `poly-example` and `realworld-example`.
- Record baseline deltas either as patch files under `baseline/patches/` or as
  explicit notes when a patch is not possible.
- Keep repository-level documentation aligned across English and Chinese
  versions.
- Do not commit host-specific paths, private endpoints, API keys, license
  blobs, or machine-local environment files.

## Validation

When changes touch the main workflow, prefer validating with:

- `uv sync --all-packages`
- `bash example/run.sh report`
- `bash example/run.sh doctor`

Add narrower smoke commands when a component cannot be fully exercised on a
fresh host.

---
> Source: [vul337/CiRCLE](https://github.com/vul337/CiRCLE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
