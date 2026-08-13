## ccgo-tree-sitter

> - Don't edit ./grammar directly. Those files are all code generated.

# Guidelines
- Don't edit ./grammar directly. Those files are all code generated.
- **Never transpile/codegen locally.** Bindings are produced only on the CI
  matrix (`mise run codegen:$GOOS-$GOARCH` on runners after workspaced place).
- Grammar **C sources** under `third-party/tree-sitter-*/` are **vendored**
  (committed). Declared in `workspaced.cue` (`#grammar`), pinned in
  `workspaced.lock.json`, refreshed with `mise run grammars:lock` /
  `mise run grammars:sync` (workspaced `core:place` — only transpile paths).
  CI packs the checkout as-is and does **not** re-run `grammars:sync` (avoids
  mass GitHub fetches / anonymous 403s). Do **not** vendor full apps
  (`third-party/tensorflow`, `third-party/django`).
- **Core tree-sitter** is a workspaced *source* only (`inputs.tree_sitter` /
  `#tree_sitter`) — not placed into the repo. Pins in `workspaced.lock.json`;
  on-disk path is the workspaced github cache (`mise run tree-sitter:path` or
  `TREE_SITTER_PATH`). Codegen resolves that path automatically.
- Codegen preprocesses with clang by default; on Windows prefer MinGW gcc on PATH.
- Do not add MSVC-header regex sanitizers; use MinGW gcc -E and ccgo ignore flags.
- `ccgo` / `libc` are normal Go module deps (forks via `replace` to
  `github.com/modernc-tree-sitter/...`), not third-party submodules.

---
> Source: [modernc-tree-sitter/ccgo-tree-sitter](https://github.com/modernc-tree-sitter/ccgo-tree-sitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
