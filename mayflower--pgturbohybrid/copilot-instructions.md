## pgturbohybrid

> - Keep C code in the existing PostgreSQL extension style: explicit memory

# Project Guidelines

## PostgreSQL Extension Style

- Keep C code in the existing PostgreSQL extension style: explicit memory
  contexts, PostgreSQL error reporting, overflow-checked size calculations, and
  no hidden heap or index access outside the access method rules.
- Do not introduce silent ABI or on-disk format changes. Any persisted format
  change needs an explicit version, magic, or compatibility check and clear
  REINDEX guidance.
- New SQL-visible behavior should have regression coverage before it becomes a
  dependency for later features.
- Do not commit generated benchmark output, regression output, local logs, or
  host-specific artifacts.

## Nix Development Environment

- Use this repository's Nix flake for builds, regression tests, TAP tests, and
  benchmark smoke checks. Do not default to the host Homebrew PostgreSQL or
  `pg_config` toolchain.
- Enter the dev environment with:
  `nix --extra-experimental-features 'nix-command flakes' develop`
- If running commands non-interactively, wrap them with:
  `nix --extra-experimental-features 'nix-command flakes' develop --command ...`
- For iterative work, use one persistent `nix develop` shell and keep using it.
  Before starting a new validation or benchmark command, first check whether a
  suitable dev shell is already running in the current session and reuse it.
  Do not repeatedly wrap every validation or benchmark command in a fresh
  `nix develop --command`; that rebuilds or re-materializes the dev closure and
  wastes time.
- At the start of a work session, verify the persistent shell once with
  `echo "$IN_NIX_SHELL"` and `pg_config --version`, then run subsequent
  commands directly inside that shell.
- For Python-only or benchmark-harness-only edits, do not refresh the Nix shell.
  Reuse the existing shell and run the Python self-checks or benchmark command
  directly.
- For long benchmark sessions, keep PostgreSQL and the benchmark shell alive
  across follow-up prompts. Do not rediscover the environment, recreate the
  shell, or restart the cluster unless the previous shell is gone, C extension
  code changed, or the user explicitly asks for a clean environment.
- For C extension changes, the running PostgreSQL process may keep the old
  shared library mapped. Refresh the Nix shell once so the dirty source is
  rebuilt into the flake-provided PostgreSQL-with-extensions wrapper, then
  restart/reset the dev cluster before SQL validation. Do not keep rediscovering
  this workflow each turn.
- Do not run plain `make install` against the flake `PG_CONFIG`; it tries to
  install into the immutable Nix store and fails with permission errors. Use the
  flake helper commands from the refreshed shell instead.
- Prefer the flake helper commands inside the dev shell:
  - `th-pg-init` to initialize/start the local PostgreSQL cluster.
  - `th-installcheck` for SQL regression tests.
  - `th-prove-installcheck` for TAP tests.
  - `th-smoke` for the minimal extension smoke test.
- Verify the environment before validation with `echo "$IN_NIX_SHELL"` and
  `pg_config --version`; the flake currently provides PostgreSQL 17. If TAP
  modules appear unavailable, first re-check that the command is running inside
  `nix develop` before treating it as a project or system issue.
- For live `pg_colbert_llama` development and validation, use the small 15m
  ColBERT pair: `VAGOsolutions/SauerkrautLM-Multi-ColBERT-15m` for PyLate
  parity and `johannhartmann/SauerkrautLM-Multi-ColBERT-15m-GGUF` for the
  PostgreSQL/llama.cpp GGUF path. Keep GGUFs under
  `.nix-dev/models/colbert-15m/`, which is ignored and must not be committed.

## Multivector / Late Interaction

- A multivector graph node is a subvector/token node, not a SQL result.
- Use these meanings consistently:
  - `nodeId`: subvector/token node.
  - `docId`: document-level identifier used for result aggregation.
  - `heaptid`: PostgreSQL heap tuple for visibility and final result output.
  - `tokenOrdinal`: token/subvector position inside the document.
- Never rank or deduplicate multivector SQL results by `nodeId`. Result ranking
  is document/heap-tuple based.
- MaxSim is similarity based: larger is better.
- SQL `ORDER BY` distance must remain smaller-is-better. For MaxSim use
  `distance = -maxsim`.
- TurboQuant may be used for approximate subvector candidate generation, but
  final MaxSim rerank should use exact float32 values unless a prompt explicitly
  changes that contract.

---
> Source: [mayflower/pgturbohybrid](https://github.com/mayflower/pgturbohybrid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
