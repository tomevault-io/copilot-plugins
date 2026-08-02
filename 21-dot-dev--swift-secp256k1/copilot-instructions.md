## swift-secp256k1

> This directory contains GitHub configuration and CI workflows.

# AGENTS.md (.github)

This directory contains GitHub configuration and CI workflows.

## Boundaries (strict)

- Do not broaden GitHub Actions `permissions` without a clear justification.
- Do not print or log secrets/tokens.
- Do not add new third-party actions without asking.

## Workflow conventions

- Preserve least privilege defaults (this repo commonly uses `permissions: {}` at workflow and job levels).
- All workflows use `env:` blocks for context values — no inline `${{ }}` interpolation in `run:` scripts.
- Avoid fragile shell output capture for UTF-8/multiline content; prefer temp files and tools like `jq` reading from files.

## Validation

- After changing workflows, run `swift test`.

---
> Source: [21-DOT-DEV/swift-secp256k1](https://github.com/21-DOT-DEV/swift-secp256k1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
