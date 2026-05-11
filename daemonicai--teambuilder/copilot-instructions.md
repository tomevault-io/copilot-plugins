## teambuilder

> This file is loaded by Claude Code when working in this repo. It captures repo-level conventions that aren't obvious from the file tree.

# Teambuilder — contributor guide

This file is loaded by Claude Code when working in this repo. It captures repo-level conventions that aren't obvious from the file tree.

## What this repo is

A collection of Claude Code skills and install scripts. There is no build step and no test suite — everything here is markdown prompts plus two install scripts (`install.sh`, `install.ps1`) served from GitHub Pages on `main`.

## Development workflow

**Always work on a feature branch. Never commit directly to `main`.**

1. Create a feature branch off `main`: `git checkout -b feature/<short-name>`.
2. Commit changes to the branch.
3. Open a PR to `main`. Get it reviewed, then merge.
4. To ship a version to users, create a GitHub Release with a tag like `v0.3.0`. The install scripts resolve the latest release automatically.

Do not merge to the `release` branch. It exists only to support users who installed before tag-based versioning and will be retired once the first tagged release has propagated.

Pinning: users can install a specific version with `VERSION=v0.2.0 curl … | bash` (see README).

## Install-time behavior to be aware of

`install.sh` / `install.ps1` clone the repo into `~/.claude/teambuilder` and **symlink** the contents of `commands/teambuild/` and each directory under `skills/` into `~/.claude/commands/` and `~/.claude/skills/` respectively.

This means:
- Anything you add to `commands/teambuild/` becomes a `/teambuild:<name>` slash command after install.
- Anything you add as a new directory under `skills/` becomes a first-class Claude Code skill after install.
- On a re-run, the install scripts delete-and-reclone the install directory when migrating between refs. Never keep local-only work in `~/.claude/teambuilder`.

## OpenSpec

This repo uses OpenSpec for non-trivial changes. Proposals live in `openspec/changes/<name>/`. Small fixes (typos, doc tweaks, single-file corrections) can skip OpenSpec and go straight to a PR.

---
> Source: [daemonicai/teambuilder](https://github.com/daemonicai/teambuilder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
