## sighthound

> `make` targets delegate to the `cargo harness` runner (`harness.rs`). Pass tunable

# CLAUDE

## Commands

`make` targets delegate to the `cargo harness` runner (`harness.rs`). Pass tunable
flags (`--min`, `--max`, `--enforce`) by invoking the runner directly, e.g.
`cargo harness crap --enforce`.

- After edits: `make check` — fix, format, lint, test, suppression report
- Pre-commit: `make pre-commit` — staged Rust files only (auto via git hook)
- Pre-push: `make pre-push` — read-only push gate: clippy + format check → acceptance → arch (auto via git hook; validates the whole pushed tree)
- CI: `make ci` — strict pipeline: clippy → format check → audit → complexity → tests → acceptance → coverage → crap → arch. CRAP is advisory (warns only — pass `--enforce` to hard-fail). Requires `uvx` on PATH.
- Complexity: `make complexity` — lizard@1.22.2 CC gate (CCN≤15, args≤8, length≤100) over src + tests
- CRAP (advisory): `cargo harness crap --max=30` — complexity × coverage gate (joins lizard --csv with `target/llvm-cov/lcov.info`). Add `--enforce` to exit 1 on offenders (default exits 0 with warning).
- Audit: `make audit` — audit dependencies for known vulnerabilities (via cargo-audit)
- Acceptance: `make acceptance` — run cucumber against `tests/features/` (warns and skips with no `.feature` files)
- Coverage: `cargo harness coverage --min=0` — cargo-llvm-cov line coverage with threshold
- Mutation (advisory): `make mutation` — cargo-mutants kill-rate on the crate
- Arch: `make arch` — cargo-modules checks against `arch.toml`
- Agents drift: `make agents-md-drift` — fail if the local CLAUDE.md mirror differs from AGENTS.md (AGENTS.md is the committed source; edit it, not CLAUDE.md)
- Sync: `make sync-agents-md` — rewrite the git-ignored CLAUDE.md mirror from AGENTS.md
- Setup: `make setup-hooks` to install git pre-commit + pre-push hooks and materialize the git-ignored agent files (CLAUDE.md mirror plus the `.claude/` + `.codex/` Stop hook wiring)
- Stop hook: auto-formats/fixes changed files, then runs complexity and CRAP (`make stop-hook`)

---
> Source: [Corgea/Sighthound](https://github.com/Corgea/Sighthound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
