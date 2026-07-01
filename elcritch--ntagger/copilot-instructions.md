## ntagger

> - `ntagger.nim`: Core executable and library; defines `Tag`, `TagKind`, and `generateCtagsForDir` plus the CLI entry.

# Repository Guidelines

## Project Structure & Module Organization
- `ntagger.nim`: Core executable and library; defines `Tag`, `TagKind`, and `generateCtagsForDir` plus the CLI entry.
- `tests/`: Unit tests (e.g., `tntagger.nim`) using Nim's `unittest`; `tests/config.nims` wires tests to the root module.
- `config.nims`: NimScript tasks (notably `test`) and local `.nimcache` configuration.
- `deps/`: Vendored dependencies and compiler sources managed via Atlas (`deps/atlas.config`).
- `nim.cfg` / `ntagger.nimble`: Local compiler settings and minimal package manifest (dependencies are expected to come from `deps/`).

## Build, Test, and Development Commands
- Install workspace deps (Atlas monorepo): run `atlas install` at the workspace root if you use Atlas.
- Build binary: `nim c ntagger.nim` (produces the `ntagger` executable in this directory).
- Run locally: `nim r ntagger.nim path/to/project` (prints a ctags-compatible tags stream to stdout).
- Run all tests: `nim test` (invokes the `test` task in `config.nims`, compiling and running all `tests/t*.nim`).
- Run a single test: `nim c -r tests/tntagger.nim`.

## Coding Style & Naming Conventions
- Indentation: 2 spaces, no tabs.
- Naming: types in `PascalCase`, procs/vars in `camelCase`, modules in `lowercase` (e.g., `ntagger.nim`).
- Exports: mark public symbols with `*` and keep the public surface small and focused.
- Formatting: use `nimpretty --backup:off ntagger.nim tests/*.nim` on changed files before sending a PR.

## Testing Guidelines
- Framework: Nim's `unittest` with clear `suite` and `test` descriptions.
- Location: add new tests under `tests/` with filenames starting with `t` (e.g., `ttagsSorting.nim`).
- Running: prefer `nim test` for the full suite; use `nim c -r tests/your_test.nim` while iterating.
- Aim to cover new branches in `generateCtagsForDir` and tag extraction logic when changing behavior.

## Commit & Pull Request Guidelines
- Commits: use short, imperative messages (e.g., "add iterator tag support"), optionally referencing issues like `(#12)`.
- PRs: include a concise summary, motivation, sample `ntagger` invocation/output if behavior changes, and a list of tests run.
- Keep diffs focused; update docs and tests alongside code changes, and avoid modifying `deps/` directly (those are managed at the workspace level).

## Security & Configuration Notes
- `nim.cfg` uses `--noNimblePath` to avoid picking up global Nimble packages; ensure dependencies are present in `deps/` instead.
- The CLI writes tags to stdout; when scripting, treat the output as untrusted text and avoid blindly eval'ing or shell-expanding it.

---
> Source: [elcritch/ntagger](https://github.com/elcritch/ntagger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
