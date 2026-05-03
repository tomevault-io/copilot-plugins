## dotfiles

> This repository is a manifest-driven dotfiles setup for macOS. Core shell config lives in `system/` and `zsh/`. Tool-specific configs live under `tools/<tool>/` such as `tools/neovim/config`, `tools/git/`, and `tools/hammerspoon/`. Custom helper scripts live in `bin/`, including the `./bin/dot` manager. The symlink manifest is `dotfiles.json`; keep it in sync whenever you add or move managed files. Template files use the `.example` suffix and pull values from `.dotenv`.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a manifest-driven dotfiles setup for macOS. Core shell config lives in `system/` and `zsh/`. Tool-specific configs live under `tools/<tool>/` such as `tools/neovim/config`, `tools/git/`, and `tools/hammerspoon/`. Custom helper scripts live in `bin/`, including the `./bin/dot` manager. The symlink manifest is `dotfiles.json`; keep it in sync whenever you add or move managed files. Template files use the `.example` suffix and pull values from `.dotenv`.

## Build, Test, and Development Commands
Use the local CLI instead of ad hoc symlink commands:

- `./bootstrap`: install base dependencies and run `./bin/dot init`.
- `./bin/dot init`: scan templates and update `.dotenv`.
- `./bin/dot link`: create symlinks defined in `dotfiles.json`.
- `./bin/dot status`: verify link health and detect conflicts.
- `./bin/dot sync --force`: pull the repo, update Homebrew and mise tools, then relink.
- `./bin/dot --help`: inspect available subcommands before extending the CLI.

## Coding Style & Naming Conventions
Match the surrounding file style instead of reformatting broadly. Python scripts in `bin/` use 4-space indentation, type hints, and standard-library-first imports. Zsh files prefer small, focused exports, aliases, and completion helpers. Lua under `tools/hammerspoon/` follows the existing tab-indented style. Keep tool configs grouped by application directory, and name new files by purpose, for example `alias.zsh`, `env.zsh`, or `path.zsh`.

## Testing Guidelines
There is no dedicated automated test suite in this repo. Validate changes with targeted checks:

- Run `./bin/dot status` after manifest or symlink changes.
- Run `./bin/dot link --force` only when you intend to overwrite conflicting local files.
- For Python script edits, use `python3 -m py_compile bin/dot bin/remote-vsc`.
- For app-specific configs, reload the owning app and confirm behavior manually.

## Commit & Pull Request Guidelines
Recent history favors short, imperative commits, often with a type or scope such as `feat(neovim): improve diffview compare workflows` or `chore(tool): change ghostty config`. Prefer `type(scope): summary` when the scope is clear; keep bulk sync commits separate from functional changes. PRs should state the affected tools or targets, note any `dotfiles.json` updates, and include manual verification steps. Add screenshots only for UI-visible changes such as Hammerspoon, Ghostty, or VS Code settings.

---
> Source: [druchtx/dotfiles](https://github.com/druchtx/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
