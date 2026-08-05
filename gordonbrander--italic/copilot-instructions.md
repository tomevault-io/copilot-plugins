## italic

> Italic is a static site generator written in Rust. Build with `cargo build`,

# Working on Italic

Italic is a static site generator written in Rust. Build with `cargo build`,
verify with `cargo test`.

## Conventions

- Modern Rust module style: `foo.rs` with a sibling `foo/` directory for
  submodules, not `foo/mod.rs`.
- User-visible behavior changes get an update to the relevant docs page.

## Documentation

The docs live in `.agents/skills/italic/references/` — they ARE the reference
material for the Italic agent skill, written agent-first: one small file per
question. The routing table in `.agents/skills/italic/SKILL.md` lists every
page; `tests/skill_docs.rs` enforces that the table stays complete, links
resolve, and troubleshooting's error strings stay verbatim with `src/`.

---
> Source: [gordonbrander/italic](https://github.com/gordonbrander/italic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
