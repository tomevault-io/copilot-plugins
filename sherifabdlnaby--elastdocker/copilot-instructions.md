## elastdocker

> Guide for AI agents working in this repo. Humans: see [README.md](README.md).

# AGENTS.md

Guide for AI agents working in this repo. Humans: see [README.md](README.md).

## What this is

A Docker Compose distribution of the Elastic Stack (Elasticsearch, Logstash, Kibana, APM, Beats). The runtime is **Docker Compose v2**; the stack is driven by **mise tasks** (`mise run up`, `mise run stack:setup`, `mise run down`, … see `mise tasks`). The `Makefile` still has the equivalent raw docker commands but is **deprecated**. There is no application source to build, the repo is compose files, service configs (YAML), Dockerfiles, and shell setup scripts.

## Toolchain (mise)

[**mise**](https://mise.jdx.dev) pins the linters/formatters, exposes tasks, and wires git hooks. `mise.toml` is the source of truth for `[tools]`, `[tasks]`, `[env]`, `[hooks]`, and `[settings]`. Don't install a linter by hand or bolt on an ad-hoc script, add a mise tool or task instead.

**Setup** (once, and per new worktree): `mise trust && mise run setup`.

**Run via mise** before calling work done:

```sh
mise run check          # all linters/formatters/validators (alias: lint); add --fix to auto-fix
mise run check --all    # whole tree (default scope is staged files; --pr = changed vs main)
mise tasks              # discover every task (up, down, logs, stack:setup, …)
mise run <task> --help  # a task's flags
```

## Git hooks (hk)

Commits run [hk](https://hk.jdx.dev) (config in `hk.pkl`), the same `check` CI runs, to lint/format staged files. Fix failures with `mise run check --fix`; don't disable steps to push a commit through. `git commit --no-verify` skips hooks for a WIP commit.

## Linters

Defined once in `hk.pkl` and shared by the `check` and `pre-commit` hooks: `shellcheck` + `shfmt` (shell), `hadolint` (Dockerfiles), `yamllint` (YAML), `taplo` (TOML), `actionlint` + `zizmor` + `pinact` (GitHub Actions lint / security / SHA-pinning), `rumdl` (Markdown), `lychee` (local/relative links, offline), `typos` (spelling), `betterleaks` (secrets), plus repo-hygiene checks (newlines, trailing whitespace, merge-conflict markers, large files, private keys, …) and `mise` self-lint.

Tunable linters keep a root config file: `.yamllint`, `rumdl.toml`, `typos.toml`, `.betterleaks.toml`, `lychee.toml`. The betterleaks step is routed to its config via `BETTERLEAKS_CONFIG` in `hk.pkl`. `.env` (placeholder defaults) is allowlisted there, not a real secret store.

## CI

- `.github/workflows/lint.yml` runs `mise run check` (`--pr` on PRs, `--all` on schedule/dispatch) and leaves one sticky comment pointing at `mise run check --fix`. Keep this green by running `mise run check --all` locally.
- `.github/workflows/smoke-test.yml` spins up the full stack (`mise run stack:setup && mise run up`), smoke-tests Elasticsearch + Kibana, and Trivy-scans the built image to the Security tab. Changes to compose files or service configs are validated here.
- `.github/workflows/auto-release.yml` drafts releases. GitHub Actions are pinned to commit SHAs (enforced by `pinact`); let `mise run check --fix` re-pin after bumping a version comment.

## Extending the setup

Changing tools, tasks, env, or hooks? Edit the config, then run `mise run check`:

- **`mise.toml`**: `[tools]` (pinned linters + hk), `[tasks]`, `[env]` (loads `.env`), `[settings]`.
- **`mise.lock`**: resolved versions + checksums. Commit it; regenerate with `mise lock` after a `[tools]` change.
- **`hk.pkl`**: the lint pipeline (add/edit a step in the shared `linters` mapping).
- **`.mise/`**: project-local state (the setup stamp is gitignored).

For tool/task/hook syntax, see the [mise](https://mise.jdx.dev) and [hk](https://hk.jdx.dev) docs.

---
> Source: [sherifabdlnaby/elastdocker](https://github.com/sherifabdlnaby/elastdocker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
