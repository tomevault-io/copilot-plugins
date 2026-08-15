## living-docs-skill

> Project guidance for any agent working in this repo. These are **hard rules**, not

# CLAUDE.md — living-docs

Project guidance for any agent working in this repo. These are **hard rules**, not
suggestions. They override default behavior. When a rule and a convenience conflict, the
rule wins.

## What this project is

`living-docs` is the deterministic layer of Living Docs authoring (see `docs/adr/0001`).
A Rust CLI owns the mechanical, template-fillable steps (`new`, `brief`, `index`,
`supersede`, `next`, `check`) so the authoring model never pays tokens for them. There is **no LLM
inside the tool** — it is deterministic by construction.

## Hard rules

### 1. No comments in code

The only permitted comments are **language docblocks** documenting a type, its params, and
its return — plus the rare non-obvious **why** (an invariant, a gotcha, a spec reference).

- Rust: `///` (item docs) and `//!` (module docs) only.
- **Forbidden:** any comment that restates *what* the code does, section banners
  (`// --- discovery ---`), TODO/FIXME left in a merged change, and commented-out code.
- If a block needs a comment to be understood, that is a signal to **extract a
  well-named function** instead of explaining it.

Rationale: this repo already regressed on decorative banners twice (lessons 3514, 3606).
Names and structure carry intent; comments drift and lie.

### 2. Self-explanatory code + complexity budget

- Intention-revealing names. Guard clauses and early returns over nesting.
- Cyclomatic ≤ 10 (≤ 8 for new functions); cognitive complexity kept low.
- Prefer deep modules with narrow interfaces over many shallow functions.

### 3. Tests assert behavior, not implementation

- Every runtime/logic change ships with tests **in the same pass**. No patch without tests.
- Tests assert observable behavior. A test that only mirrors the implementation is a smell.
- The fitness functions in ADR 0001 (`new` output passes `check`; `index` is idempotent;
  `supersede` leaves both records linked and conformant) stay green.

### 4. Determinism boundary

The tool never writes rationale prose, never chooses a doc's epistemic type, never resolves
which alternative wins. Those belong to the authoring model. Everything the tool does must
be reproducible from its inputs.

### 5. Responsibility split + file-size ratchet (issue 0028)

Maintainability is a gate, not an advisory. Layout rules for all Rust code:

- **One responsibility per module.** `cli/src/main.rs` holds only clap wiring and
  dispatch; every verb lives in `cli/src/commands/<verb>.rs`. New verbs are BORN in
  their own module — never added to `main.rs`.
- **Hard cap: 30 lines per function.** Enforced by clippy `too_many_lines`
  (`clippy.toml` threshold 30, `-D warnings` in CI). This is the primary clarity
  instrument — Clean Code's limit is about functions, not files.
- **Hard cap: 300 lines per `.rs` file** (sibling `tests.rs` has its own 300 cap).
  Enforced by `scripts/check-file-size.sh` (ratchet: a grandfathered file may only
  shrink; growing one fails the check).
- **Sibling test files.** A `#[cfg(test)]` module over ~100 lines moves to
  `<module>/tests.rs` via `mod tests;` — private access preserved, production file clean.
- Deep modules still win over many shallow files (rule 2): split by responsibility,
  never by line-count alone — the file cap is the backstop, not the design driver.

## Architecture

Target shape is a **modular monolith** (start here; split into crates only when a real
seam demands it), organized hexagonally:

```
living-docs-core   — domain + ports (traits), no I/O
    ports:  DocStore (read/write records) · SearchIndex (FTS5)
adapters:
    fs-store   (LocalFileStorage)  → .md files
    db-store   (DatabaseStorage)   → SQLite normalized + FTS5
fronts:
    cli   → depends on core, injects an adapter
    web   → axum server on core, reads the db-store projection
```

### Locked decisions

- **Single repository, Cargo workspace (monorepo).** `core`, `cli`, and `web` share one
  domain and ship together. They live as members of one workspace, not separate repos:
  domain changes stay atomic (one PR, one CI), with no cross-repo version coordination.
  The hexagonal ports are the extraction seam — splitting into separate repos later is
  cheap *because* the boundary already exists. **Reconsider only when** a front needs an
  independent deploy cadence or separate ownership; until then, splitting adds release
  friction for no gain.
- **Config-selected, mutually exclusive backends (ADR 0003).** `fs-store` and `db-store`
  sit behind `DocStore`, chosen by config/flag — never both at once. Exactly one backend is
  authoritative per deployment (file-mode: `.md` in git; db-mode: the database), so there is
  **no bidirectional sync and no source-of-truth conflict to resolve** — that framing was
  considered and explicitly rejected. ADR 0016 layers Atlas's write path on top of this:
  browser authoring is db-mode-only, gated by a per-record `revision` optimistic-concurrency
  precondition, never a cross-backend merge.
- **Web = Rust/axum reusing `living-docs-core`.** One language, one build, no model drift
  between CLI and web. Web reads the db-store projection.
- **CLI search defaults to the DB backend**, FTS5-powered (`living-docs search "..."`),
  with an explicit sync step to (re)build the projection from records.
- **Delivery sequence:** S1 extract `living-docs-core` + ports (refactor, no new behavior)
  → S2 `db-store` + FTS5 + `search` → S3 web. Each slice is vertical and demoable.

## Working conventions

- Every architectural fork (adapter sync contract, DB schema, web surface) gets an ADR via
  `living-docs new adr "..."` **before** code. Decide, then implement.
- `living-docs check` must pass over `docs/` — it is the doc-gate.
- **Docs authoring is CLI-first and gate-enforced (ADR 0021).** Write ONLY the body below
  the closing `---` of a record; numbering, frontmatter, supersede links, and index rows
  come from `living-docs new`/`status`/`supersede`/`index`/`fmt`. A PreToolUse hook
  (`.claude/settings.json` → `skills/living-docs/hooks/block-docs-handwrite.sh`) blocks
  hand-writes under `docs/{adr,bdr,prd,issues}/`, and `.githooks/pre-commit` runs the
  doc-gate before every commit.
- Conventional Commits; ticket ID when one exists. No AI attribution in commit messages.
- Never bypass a failing hook with `--no-verify`; fix the cause.

---
> Source: [ejklock/living-docs-skill](https://github.com/ejklock/living-docs-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
