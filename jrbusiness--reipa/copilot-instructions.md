## reipa

> Use **ripgrep (`rg`)**, not `grep`, for searching this repository.

# Agent conventions

## Searching the codebase

Use **ripgrep (`rg`)**, not `grep`, for searching this repository.

`rg` respects `.gitignore` (so it skips `target/`, build artifacts, and the
large binaries under `scratchpad/`), is much faster on this tree, and has
saner defaults. Reach for it in scripts, ad-hoc searches, and CI.

- Search: `rg pattern` (recursive by default) instead of `grep -rn pattern .`
- Filter by type: `rg -t rust pattern` / `--glob '*.rs'`
- If a script must run where `rg` may be absent, install it or gate on
  `command -v rg` before falling back — do not default to `grep`.

---
> Source: [JRBusiness/REipa](https://github.com/JRBusiness/REipa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
