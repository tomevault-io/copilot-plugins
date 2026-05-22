## dotfiles

> This file is for coding agents working in this repository.

# AGENTS Guide for `dotfiles`

This file is for coding agents working in this repository.
It captures practical commands and style conventions observed in the codebase.

## Repo Scope

- This is a personal dotfiles repo for macOS shell/editor/tooling setup.
- Main areas:
  - `zshrc`, `extra_shell_functions.sh`
  - `gitconfig`
  - `ghostty_config`
  - `chezmoi/dot_config/opencode/`
  - `astronvim/` (Lua-based Neovim config)
- There is no app build pipeline in this repository.
- There is no formal automated test suite in this repository.

## Rule Files Check

- Checked for Cursor rules:
  - `.cursorrules` (not present)
  - `.cursor/rules/` (not present)
- Checked for Copilot rules:
  - `.github/copilot-instructions.md` (not present)
- Action: no extra agent rule files need to be merged beyond this document.

## Environment and Tooling

- Primary language in `astronvim/`: Lua.
- Shell files: POSIX shell + zsh usage.
- Formatting/lint config files: `astronvim/.stylua.toml`, `astronvim/selene.toml`, `markdownlint-cli2.jsonc`.
- Lua language config files: `astronvim/neovim.yml`, `astronvim/.luarc.json`, `astronvim/.neoconf.json`.
- Mason installs `stylua`, `shellcheck`, `markdownlint-cli2`, plus language tools for Go/Python/Rust/YAML/Bash.

## Setup / Bootstrap Commands

- Legacy manual symlink notes live in `links` (deprecated).
- Install Homebrew packages via split Brewfiles when using chezmoi:
  - `brew bundle --file ~/.Brewfile.base`
  - `brew bundle --file ~/.Brewfile.personal` or `~/.Brewfile.work`
- Start Neovim once to bootstrap plugins: `nvim`.

### Chezmoi-based Setup

- Preferred distribution path uses `chezmoi` source state in `chezmoi/`.
- Initialize chezmoi with this source:
  - `chezmoi init /Users/tomgoren/dev/dotfiles/chezmoi`
- Apply managed config:
  - `eval "$(op signin --account my.1password.com)" && chezmoi apply` (required when templates resolve `op://` secrets)
- Bootstrap script run by chezmoi (once):
  - `chezmoi/run_once_after_10_bootstrap.sh.tmpl`
- Homebrew package sets are stored as:
  - `Brewfile.base`, `Brewfile.personal`, `Brewfile.work`
  - `chezmoi/dot_Brewfile.base`, `chezmoi/dot_Brewfile.personal`, `chezmoi/dot_Brewfile.work`

## Build / Lint / Test Commands

Because this is a dotfiles repo, prefer validation + lint over "build".

### Build-like / Validation Commands

- Validate Neovim config can start headless:
  - `nvim --headless '+qa'`
- Trigger plugin sync/update in headless mode:
  - `nvim --headless '+Lazy! sync' '+qa'`

### Lint / Format Commands

- Lua formatting (AstroNvim config):
  - `stylua astronvim/lua`
- Lua static analysis:
  - `selene astronvim/lua`
- Shell lint:
  - `shellcheck zshrc extra_shell_functions.sh`
- Markdown lint:
  - `markdownlint-cli2 '**/*.md'`

### Single-file / Targeted Lint Commands

- Single Lua file format:
  - `stylua astronvim/lua/plugins/lint.lua`
- Single Lua file lint:
  - `selene astronvim/lua/plugins/lint.lua`
- Single shell file lint:
  - `shellcheck zshrc`
- Single markdown file lint:
  - `markdownlint-cli2 README.md`

### Test Commands (Current State)

- There is no dedicated test runner (no `pytest`, `go test`, `cargo test`, etc.).
- "Run a single test" is currently not applicable.
- Closest equivalent for targeted verification is single-file lint/format commands above.
- For behavioral checks, use Neovim headless startup as smoke test (`nvim --headless '+qa'`).

## Code Style Guidelines

Follow existing local patterns first; these are repository conventions.

### Global Principles

- Keep changes minimal and specific to requested behavior.
- Preserve existing file structure and plugin organization.
- Prefer readability and explicit intent over clever abstractions.
- Do not introduce new frameworks or tooling without strong reason.

### Lua Style (AstroNvim)

- Use 2-space indentation.
- Keep line length around 120 columns.
- Use Unix line endings.
- Let StyLua manage quote normalization (`AutoPreferDouble`).
- Follow existing call-parentheses style from StyLua config (`call_parentheses = "None"`).
- Keep module files returning a single spec/config table when applicable.
- Keep `---@type LazySpec` / related annotations when already used.
- Keep plugin config split by concern in `astronvim/lua/plugins/*.lua`.

### Imports / Module Loading (Lua)

- Prefer `local x = require "module"` when reused multiple times.
- Inline `require("module").fn()` is acceptable for one-off calls.
- Use existing AstroNvim import table conventions (`{ import = "community" }`, `{ import = "plugins" }`).
- Do not reorder imports/spec entries unless behavior depends on it.

### Naming Conventions

- File names: lowercase, descriptive (`astrocore.lua`, `lint.lua`).
- Locals: short but meaningful (`lint`, `md`, `cfg`, `opts`).
- Keymaps: include clear `desc` values.
- Keep naming consistent with upstream AstroNvim and plugin APIs.

### Error Handling

- For startup-critical operations, follow pattern used in `astronvim/init.lua`:
  - check command result/error code
  - display explicit message via `vim.api.nvim_echo`
  - stop execution safely when bootstrapping fails
- Prefer defensive checks before mutating optional plugin config fields.
- Avoid swallowing failures silently.

### Configuration Patterns

- Respect disabled template guards:
  - `if true then return {} end`
  - `if true then return end`
- If enabling a template file, remove guard and prune unused examples.
- Extend defaults instead of replacing when possible (e.g., deep-extend/list-insert patterns).

### Shell Style (`zshrc`, `extra_shell_functions.sh`)

- Use `[[ ... ]]` for shell conditionals.
- Quote variable expansions and command substitutions unless intentional globbing is required.
- Keep helper functions small and task-focused.
- Preserve safe-guard behavior on risky git flows (e.g., branch checks before push).

### Markdown Style

- Repository markdown lint config disables only `MD013` (line length).
- Keep headings and bullets simple and readable.
- Prefer fenced code blocks with language tags where useful.

## Agent Workflow Recommendations

- Before edits, read nearby files for local conventions.
- After edits, run targeted lint for touched files first.
- For `astronvim/` changes, run headless Neovim smoke check.
- Avoid touching unrelated dotfiles in the same change.
- If adding new tools/commands, document them in this file.

---
> Source: [tomgoren/dotfiles](https://github.com/tomgoren/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
