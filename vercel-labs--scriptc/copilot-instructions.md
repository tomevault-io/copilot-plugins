## scriptc

> Guidance for agents (and humans) working on this repository. These conventions apply repo-wide; the docs site under `docs/` additionally has its own conventions in `docs/AGENTS.md`.

# Agent Guide

Guidance for agents (and humans) working on this repository. These conventions apply repo-wide; the docs site under `docs/` additionally has its own conventions in `docs/AGENTS.md`.

## Build and test

```bash
pnpm install && pnpm -r build      # build the workspace
SCRIPTC_TEST_WORKERS=4 pnpm test   # plain lane: differential corpus + diagnostics snapshots
SCRIPTC_SAN=1 pnpm test            # sanitized lane: the same corpus under ASan + refcount audit
```

Both lanes green is the bar before shipping any change. `SCRIPTC_TEST_WORKERS` caps the vitest worker pool so concurrent agents don't contend for cores; full-suite runs also queue behind an advisory lock per lane.

Corpus programs are differential tests against Node: every program runs under Node and as a compiled native binary, and stdout, stderr, and exit codes must match byte-for-byte. A new feature lands with corpus programs that pin its behavior both ways.

## Where things live

- `packages/compiler` — the frontend (tsc API to IR), the typed IR with validator and serializer, and the LLVM and C backends.
- `packages/runtime` — the C runtime compiled into every scriptc binary.
- `packages/cli` — `scriptc build | run | coverage`.
- `tests/` — the differential corpus, diagnostics snapshots, and the harness.
- `docs/` — the documentation site (standalone pnpm workspace); see `docs/AGENTS.md`.
- `scripts/` — repo tooling, including the release version stamp.

## Releases

Releases are maintainer-run; see [RELEASING.md](./RELEASING.md).

---
> Source: [vercel-labs/scriptc](https://github.com/vercel-labs/scriptc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
