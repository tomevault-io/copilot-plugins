## flex-x

> This repository contains a small Emacs Lisp completion package. Source files

# Repository Guidelines

## Project Structure & Module Organization

This repository contains a small Emacs Lisp completion package. Source files
live at the repository root:

- `flex-x.el`: completion style implementation, matching, sorting, highlighting,
  metadata adjustment, and optional extra matchers.
- `flex-x-tests.el`: ERT coverage for matching semantics, sorting, highlighting,
  text properties, and compatibility paths.
- `FEATURES.md`: required behavior, explicit non-goals, and feature-scoped
  simplicity constraints.
- `README.org`: user-facing setup and behavior.

Compiled `.elc` files may exist in the tree, but contributors should edit the
`.el` sources and regenerate bytecode only when intentionally updating compiled
artifacts.

## Build, Test, and Development Commands

- `make test`: run the ERT suite in batch Emacs.
- `make compile`: byte-compile sources and tests with warnings treated as
  errors.
- `make load`: perform a minimal load check in a clean Emacs session.
- `make package-lint`: run `package-lint` on the main package file.
- `make checkdoc`: check docstrings and documentation conventions.
- `make check-declare`: validate external declarations.
- `make check`: run the full validation set above.
- `make clean-elc`: remove generated `.elc` files and the temporary compile
  directory.

Use Emacs 30.1 or newer, matching the `Package-Requires` header in `flex-x.el`.
Use `EMACS=/path/to/emacs make check` to validate against a specific Emacs
binary.

## Coding Style & Naming Conventions

Use standard Emacs Lisp formatting and keep `lexical-binding: t` in source
headers. Public package symbols should use the `flex-x-` prefix; private helpers
should use `flex-x--`. Test helpers should use `flex-x-tests--`, and ERT test
names should begin with `flex-x-` and describe behavior, for example
`flex-x-extra-pattern-function-is-cached-per-term`.

Prefer clear docstrings for public custom variables and functions. Keep comments
focused on non-obvious completion behavior, scoring, caching, or compatibility
with Emacs internals.

## Testing Guidelines

Tests use the built-in `ert` framework. Add or update tests in `flex-x-tests.el`
for every behavior change, especially matching semantics, sorting,
highlighting, text properties, and compatibility paths for different Emacs
completion internals. Keep tests deterministic by binding package customization
variables locally with `let`.

Run `make test` for focused validation and `make check` before submitting
broader changes.

## Commit & Pull Request Guidelines

Recent history uses short, direct Japanese commit summaries such as
`vertico-buffer-frameの設定を調整` and `:afterキーワードを追加`. Follow that concise
style unless the project adopts a new convention. Keep commits scoped to one
logical change.

Pull requests should include a brief description, the affected completion
behavior, and the exact commands run for verification. Link related issues when
available, and include screenshots only for UI-visible highlighting changes.

## Agent-Specific Instructions

Do not rewrite unrelated Emacs configuration files outside this package while
working here. Preserve existing user changes in the wider `.emacs.d` worktree.

Before making non-trivial code changes, read `FEATURES.md` and keep the
implementation limited to the listed behavior. When adding or changing behavior,
update `FEATURES.md` first or in the same change so the required functionality
remains explicit.

Implement the simplest maintainable code that satisfies `FEATURES.md`. Do not
add abstractions, options, compatibility paths, optimizations, or unrelated
refactors unless they are needed for a listed feature. Treat performance
requirements as features, and document the reason or target when an optimization
adds complexity.

---
> Source: [kn66/flex-x](https://github.com/kn66/flex-x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
