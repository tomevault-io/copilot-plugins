## c-code-style

> This repository defines a C coding style. If you (an AI coding agent) are

# Agent instructions

This repository defines a C coding style. If you (an AI coding agent) are
writing, editing, or reviewing `.c`/`.h` files in a project that adopts this
style guide, follow this file.

## Source of truth

Read [`CODING_RULES.md`](CODING_RULES.md) before writing code — it is the authoritative,
complete rule set. Do not assume "typical" C conventions; several rules here
are deliberately non-default (e.g. `(void)` required on zero-parameter
functions, at most 2 `return` points per function, `stdbool.h` avoided,
3-character minimum name length). `template.c` and `template.h` are
worked, clang-format-verified reference examples — match their formatting
and doxygen alignment exactly.

## What the formatter fixes for you

`clang-format` (this repo ships `.clang-format`) handles indentation,
spacing, brace placement, pointer-asterisk alignment, function-prototype
alignment, and trailing commas in struct initializers. Don't hand-format
these — write reasonably and run the formatter afterward (see below).

## What you must apply yourself while writing

A formatter cannot infer these; apply them as you write, not after:

- Names (variables, parameters, struct/enum members): lowercase, underscore
  word-separated, **at least 3 characters**. No `__`/`_` prefix (reserved).
  `static` module-private functions get a `prv_` prefix.
- `stdint.h` types except `char`/`float`/`double`; avoid `stdbool.h` (use
  `1`/`0`).
- Don't rely on static initializers for `global`/`static` variables needing a
  real startup value in embedded contexts — add a module `init()` function.
- All locals declared at the top of a block, grouped by type, ordered:
  structs/enums (and pointers) → integers (widest unsigned first) → floating
  point.
- `const`-correct pointers/parameters; generic pointers are `void*`, cast
  internally, never cast away a `void*` return value.
- Every zero-parameter function uses `(void)`, never empty `()`.
- At most 2 `return` points per function: one early return for
  parameter/argument validation, one at the end. No `return` in the middle —
  use a return-value variable instead.
- Check pointers against `NULL` before dereferencing (unless provably
  non-`NULL` by construction). Set a pointer to `NULL` immediately after
  `free()`.
- Boolean-treated variables: `if (x)` / `if (!x)`, never `== 1`/`== 0`.
  Counter/value variables: `> 0` / `== 0`, never bare `if (x)`.
- Doxygen on every function (including `static`): `\brief`, `\param[in]` /
  `\param[out]` per parameter, `\return` if non-`void`. Written at the
  function's **definition** (the `.c` file body), never duplicated at the
  header declaration. Description text aligned to column 22 (match
  `template.c`). Backtick any identifier/constant/literal in prose.

## Format and verify

After editing any `.c`/`.h` file, run:

```sh
scripts/check-format.sh <changed-file.c> [<changed-file.h> ...]
```

It checks `clang-format` is installed (>= version 20; prints the right
install command for your OS and exits non-zero if not, or if too old), then
runs `clang-format -i` using this repo's `.clang-format`.

To verify without writing changes:

```sh
scripts/check-format.sh --check <file.c> [<file.h> ...]
```

Exits `0` if compliant, non-zero with the specific violations otherwise. This
script and flow were verified directly in this repository (missing-tool path,
detect → fix → re-verify round trip, and both template files passing
`--check` as-is).

---
> Source: [MaJerle/c-code-style](https://github.com/MaJerle/c-code-style) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
