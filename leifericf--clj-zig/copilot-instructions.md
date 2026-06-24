## clj-zig

> The design lives in `docs/` (start at `README.md`); the design

# clj-zig agent guide

The design lives in `docs/` (start at `README.md`); the design
principles are in `docs/07-design-principles-and-decisions.md` and the
decisions are ADRs in `docs/adr/`. This file covers only how to work,
not what clj-zig is.

The project is pre-code. The dossier is the spec; the skills below are
the discipline.

## What shapes the code

- Clojure-first: clj-zig is a Clojure library. The Zig is generated
  wrapper source plus the user's body.
- Functional core, imperative shell (ADR 16): parsing, normalization,
  spec construction, and source generation are pure; compiling,
  caching, loading, and Var rebinding are the shell.
- The boundary contract is data, and generated Zig, specs, and
  diagnostics stay inspectable.

## Skills

- Writing code: `write-clj`, `write-zig`, `write-tests`. Read the
  matching standard first:
  `.claude/skills/check-style/references/{clj,zig,prose}-style.md`.
- Reviewing: `check-style`. One bug: `fix-bug`. A whole change:
  `implement-change`.
- A correction mid-task: `capture-guidance`, then keep working;
  promote later with `incorporate-feedback`.
- A real choice between alternatives: `record-decision`, which writes
  an ADR to `docs/adr/`.

## Tooling

- Clojure CLI and `deps.edn`, not Leiningen. Tests are `clojure.test`
  under the project's `:test` alias once the skeleton lands.
- REPL-driven development is the primary loop. `defnz` redefinition is
  a core feature; exercise it at the REPL, not only in tests.
- The native side shells out to `zig`; `zig fmt` owns Zig formatting.

## Commits

Single line, category first: `Category: Imperative subject`. Capital
first word, no trailing period, within 70 characters, effect not diff.
No body, no `Co-Authored-By` or other attribution, no version numbers,
no em dashes.

---
> Source: [leifericf/clj-zig](https://github.com/leifericf/clj-zig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
