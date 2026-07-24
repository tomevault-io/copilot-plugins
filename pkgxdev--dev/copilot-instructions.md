## dev

> Public repository for developer environment activation tooling.

# AGENTS: dev

Public repository for developer environment activation tooling.

## Core Commands

- `deno fmt --check .`
- `deno lint .`
- `deno check ./app.ts`
- `deno test --allow-read --allow-write --allow-env`

## Always Do

- Keep activation behavior deterministic across shells and platforms.
- Preserve compatibility with `pkgm` and `libpkgx` interactions.

## Ask First

- Changes to GitHub Action interfaces.
- Changes to environment variable behavior with broad downstream impact.

## Never Do

- Never introduce hidden global side effects during activation.
- Never weaken environment isolation guarantees.

---
> Source: [pkgxdev/dev](https://github.com/pkgxdev/dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
