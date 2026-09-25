## code

> `AGENTS.md` is the canonical instruction file for this repo. Read it first. This file exists for tools that still load `CLAUDE.md`.

# CLAUDE.md - Noumena Code (OSS)

`AGENTS.md` is the canonical instruction file for this repo. Read it first. This file exists for tools that still load `CLAUDE.md`.

## Source Of Truth

- Read `AGENTS.md` first, then this file.
- If `CLAUDE.md` and `AGENTS.md` disagree, follow `AGENTS.md`.
- Do not add new repo policy here unless it is also reflected in `AGENTS.md`.

## What We Are Building

Noumena Code is the open-source AI coding assistant: a terminal-native agent that edits code, runs commands, and helps carry multi-step development workflows. This repo is the standalone public source export of `code/` from Noumena's internal monorepo.

## Repo Emphasis

- Bias toward product reliability, build correctness, contributor clarity, and OSS safety.
- GPU training, kernels, and inference hot-path work belong in `noumena`.
- Prefer explicit, documented build/test entrypoints over hidden or ambient state.
- Do not leak internal hosts, IPs, or unreleased model identifiers into public files.

## Operating Contract

- Print `Mode: No-Edits` or `Mode: Execution` at the top of every response when applicable.
- Before executing, confirm repo root with `git rev-parse --show-toplevel` and working copy state with `git status --short`.
- Before editing, list the exact files you will modify.
- Do not touch out-of-scope files.
- Verify with files, diffs, logs, and commands. Do not guess.
- This is a Git repo. Do not use Sapling commands (`sl`, `sl status`, etc.).
- Do not run destructive Git operations without explicit approval.
- Never `git push --force` to `main`.
- Do not install system packages, global packages, or modify shell configs as a fix. If the repo needs a new dependency, add it to `package.json`/`Cargo.toml`.
- The canonical build is `bun run build`. The canonical test runner is `bun test <file>` or `bun run test`.
- Start validation with the smallest relevant test file, then broaden only as needed.

## Repeated Corrections To Internalize

- This is a **Git** repo, not Sapling. Use `git`, not `sl`.
- The supported build/test stack is **Bun + Rust/Cargo**. Do not assume Python, Docker, Kubernetes, or Buck2 are available.
- Do not run `cargo build` manually at the root; native modules are built by the Bun build scripts.
- Do not delete `.tmp/` or `dist/` "just to be safe".
- Do not add internal endpoints, IPs, or unreleased model identifiers to public code or tests.
- Local-only workarounds are not the supported answer. If a fix requires a new dependency or system tool, document it in `README.md` and add it properly.

## Hard Anti-Patterns

- Do not modify the user's shell config (`.bashrc`, `.zshrc`, `.profile`).
- Do not install global packages (`npm install -g`, `cargo install`, `apt install`, etc.) as a fix.
- Do not push directly to `main` for non-trivial changes; open a PR.
- Do not add placeholder JSON, mock-only routes, `unimplemented!()`, or silent fallbacks as progress.
- Do not rely on internal-only infrastructure in public code unless gated by `isInternalBuild()` or a documented env var.

## Build And Validation Quick Commands

```bash
cd /mlstore/src/noumena-network/code
bun install
bun run build
bun test src/path/to/file.test.ts
bun run test
```

## Git Quick Commands

```bash
cd /mlstore/src/noumena-network/code
git rev-parse --show-toplevel
git status --short
git diff -- <paths>
git log --oneline -3
```

## References

- `AGENTS.md` is canonical.
- `README.md` is the public user/contributor guide.
- `OSS_BUILD.md` documents build modes and supported targets.

---
> Source: [Noumena-Network/code](https://github.com/Noumena-Network/code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
