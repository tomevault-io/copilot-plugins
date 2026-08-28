## hotcoco

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

hotcoco is a pure Rust port of [pycocotools](https://github.com/ppwwyyxx/cocoapi) with PyO3 Python bindings. It provides 13-23x speedups over pycocotools for bbox, segmentation, and keypoint evaluation.

- **Primary language:** Rust. All core logic lives in `hotcoco`.
- **Python bindings:** PyO3/maturin in `hotcoco-pyo3`, exposed as the `hotcoco` Python package.
- **CLI:** `hotcoco-cli` binary wrapping the Rust library.

### Key Architecture

- Single root `pyproject.toml` acts as both maturin build config and Python package definition; `manifest-path = "crates/hotcoco-pyo3/Cargo.toml"` points maturin at the cdylib. `[tool.uv] package = false` means uv won't auto-build — always use `just build` explicitly.
- `hotcoco-pyo3` uses `hotcoco-core` as the Cargo dependency alias for `hotcoco` to avoid name collision with the `hotcoco` Python module name
- Python bindings return plain dicts (not wrapped Rust structs) matching pycocotools conventions
- Mask operations handle numpy row-major <-> Rust column-major transposition in the PyO3 layer
- `cargo build --workspace` will fail at link time for hotcoco-pyo3 (expected — cdylib needs Python). Use `cargo check` instead, or build via maturin.
- **Type stubs:** `python/hotcoco/__init__.pyi` is hand-written and must be updated whenever the Python API changes (new methods, renamed parameters, changed return types). Run `uv run pytest scripts/test_stubs.py` to check for drift. The test catches missing names but not signature mismatches — review the stub manually when changing signatures.

## Metric Parity

All COCO evaluation metrics must match pycocotools: 12 for bbox/segm, 10 for keypoints (no small area range), 13 for LVIS (adds APr/APc/APf/AR@300).

- **Always ensure exact parity when modifying evaluation logic.** Run `cargo test` after Rust changes.
- Verified on val2017: keypoints exact, bbox within 0.0001, segm within 0.0002.
- When in doubt, run differential tests against pycocotools on real COCO data before declaring a task complete.
- After any change to evaluation logic, run `/parity` — it holds the full verification sequence and the expected tolerances.

## Testing

- Run `cargo test` after any Rust code changes and verify all tests pass before committing.
- For Python binding changes: `just build` as a smoke test, then `just parity` to verify metrics.
- `just test` runs `cargo test` + fast Python regression tests (`scripts/test_parity.py`) — safe for CI, completes in under 30s.
- `just fuzz` runs the hypothesis-based fuzzer (`scripts/fuzz_parity.py`) — use to hunt for parity bugs, not in CI. Takes several minutes.
- Model: use the fuzzer to *find* bugs, then prove fixes with Rust integration tests in `crates/hotcoco/tests/`.

## Tool Preferences

- **Always use `uv run python` — never bare `python` or `python3`.** This project uses uv-managed Python; the OS Python is not the project environment.
- For library/crate documentation, prefer a docs MCP server (e.g. context7) when one is connected. None is configured by default, so WebFetch against docs.rs, python.org, and PyPI is the working fallback — those domains are already in the project allowlist.

## Build Gotchas

`just` is the task runner — run `just --list` for the current recipes. The non-obvious parts:

- **`uv sync` alone is not enough.** Without `--all-extras` it skips `maturin`, and `just build` then fails. Always use `just setup` for first-time setup.
- `uv run python` works from anywhere in the repo (no need to cd first).
- The `coco` CLI is installed into `.venv/bin/coco` by `just build`. Run it as `uv run coco <subcommand>` (or activate the venv with `source .venv/bin/activate` for bare `coco`).

## Documentation

- This project targets Python users first, Rust users second. Documentation, README, and examples should lead with Python usage in a Python-first tone similar to Polars. Do not be Rust-centric.
- Before making large-scale changes (docs revamps, major refactors), present a concrete preview or small example for approval first. Do not rewrite everything at once. For small additions (a single new page, a new section), just write it directly.

Docs are built with Zensical (config: `zensical.toml`). Preview locally with `zensical serve`.

When updating documentation (`docs/`) or `README.md`, always ensure both reflect the same information. Any change to one must be checked against the other — benchmark numbers, API examples, CLI flags, installation instructions, and feature descriptions must stay consistent across both.

## Design System ("Cold Brew")

All visual surfaces (browse UI, docs site, matplotlib, Plotly dashboard) share the **Cold Brew** theme. The canonical spec — colors, fonts, the 10-color chart palette, and which file owns each token — is the `cold-brew` skill. Consult it before changing any colors, fonts, or chart palettes, and when adding a new visual surface. Never eyeball new values.

## Pre-Commit Checks

A git pre-commit hook in `.github/hooks/pre-commit` runs formatting, clippy, and tests. All must pass or the commit is rejected.

To install the hook (one-time setup — works in both main repo and worktrees):

```bash
git config core.hooksPath .github/hooks
```

If formatting fails, run `cargo fmt --all` to fix, then re-commit. If clippy fails, fix the warning before committing. **Never suppress clippy warnings with allows. Never skip the hook with `--no-verify`.**

## Git Workflow

- **Never commit or push unless explicitly asked.** Wait for the user to say "commit", "push", or "ship it" before running any git commit/push commands.
- When committing and pushing, always verify the current git status first to avoid trying to commit already-committed changes. Check `git status` and `git log --oneline -3` before any commit/push operation.
- Keep commits clean: never include build artifacts, compiled files, or `__pycache__` directories. Review staged files carefully before committing. If unsure, ask before committing.
- Commit message body: use bullet points, not prose paragraphs.
- Main branch: `main`.
- **Standard pre-commit sequence:** `/simplify` → `/review` → `/ship` → `/commit`. Run all four in order when a feature is done.
- `/ship` is the gate for communication surfaces — CHANGELOG, ROADMAP, docs sync, and parity. Do not commit and then update docs/CHANGELOG after; update everything first, then commit once.

### Worktrees

Sessions may run in a git worktree (e.g. `.claude/worktrees/<name>/`). Worktrees share the same `.git` history but have their own branch and working directory.

- **Commits land on the worktree branch**, not `main`. When the user says "push", they likely mean push to `main`. Use `git -C <main-repo> cherry-pick <hash>` then push from the main repo, or ask to confirm.
- **`data/` is gitignored** and won't exist in a fresh worktree. Scripts that need COCO data (`parity.py`, `bench.py`) require a symlink to the main repo's `data/`.
- **`.venv`** is created per-worktree by `just setup`. Run it once in a new worktree.
- **Pre-commit hook** uses `git config core.hooksPath` set to the *relative* path `.github/hooks`, so it resolves in both the main repo and worktrees. Verify with `git config core.hooksPath` — it must print `.github/hooks`. An absolute path, or a symlink under `.git/hooks`, works in the main repo but is not worktree-portable; re-run the setup command in Pre-Commit Checks to correct it.
- **All paths in skills and scripts must be relative** — never hardcode the main repo path.

---
> Source: [derekallman/hotcoco](https://github.com/derekallman/hotcoco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
