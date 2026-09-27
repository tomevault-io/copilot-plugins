## goverify

> Front door for AI coding agents working on goverify.

# AGENTS.md

Front door for AI coding agents working on goverify.

## Orientation

- Design spec (source of truth): `docs/superpowers/specs/2026-07-16-goverify-design.md`
- Crate boundaries and why: `ARCHITECTURE.md`
- Security stance: `docs/threat-model.md`

## Workflows — always via mise tasks

Run `mise tasks` to list them. The ones you'll need: `build`, `test`,
`lint` (blocking-tier static checks), `fmt`, `corpus` (full
extractor→.gvir pipeline + determinism suite), `audit`, `secrets`,
`proto-gen` (regenerate Go protobuf bindings — commit the output),
`shakeout` (manual: `goverify check` over pinned bbolt, not part of the
blocking gate). Blocking CI runs `lint` + `test` (which includes the
corpus determinism suite) plus `secrets` + `audit`; run those before
pushing.

## Rules that will bite you

- **Determinism is the root invariant.** Identical source bytes must
  produce byte-identical `.gvir`: no timestamps, no absolute paths, no
  map-iteration order reaching output. Sort before emitting; the corpus
  suite enforces this.
- The **only** Go code lives in `extractor/`. Everything else is Rust.
- The `.gvir` schema is single-sourced in `proto/gvir/v1/gvir.proto`.
  After changing it: `mise run proto-gen`, bump `schema_version` +
  `SCHEMA_VERSION` (Rust) + `schemaVersion` (Go) together, commit the
  generated `extractor/gvirpb/`.
- Dependencies are deliberately few (see the design spec §13). Adding a
  crate needs justification, and `Cargo.lock` is committed.
- Parsers of bytes the analyzer didn't write must reject, never panic
  (fuzz targets in `fuzz/`).
- Errors degrade, never die: skip with a diagnostic and keep analyzing.

---
> Source: [rahulmutt-ava/goverify](https://github.com/rahulmutt-ava/goverify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
