## nvim-config

> Operational guide for coding agents working in this repository.

# AGENTS.md

Operational guide for coding agents working in this repository.

## Repository Scope

- Root repo contains helper scripts (`install.sh`, `setup.sh`) and a Neovim config in `nvim/`.
- Main code is Lua-based Neovim configuration.
- There is currently no dedicated CI pipeline or formal test suite in this repo.

## Rule Files Discovery

- Cursor rules: none found (`.cursorrules` and `.cursor/rules/` absent).
- Copilot rules: none found (`.github/copilot-instructions.md` absent).
- If these files are added later, treat them as higher-priority local instructions.

## Working Directory Guidance

- For Neovim config edits, use `nvim/` as your primary working directory.
- For repo-wide docs/scripts, work from repository root.
- Do not move the Neovim config out of `nvim/`.

## Environment and Setup Commands

- Symlink config into Neovim default location:
  - `./setup.sh`
- Install pinned Neovim version helper (v0.11.5 currently):
  - `./install.sh`
- Verify Neovim is available:
  - `nvim --version`

## Build Commands

There is no compile/build artifact step for this repo. Use these as practical "build/smoke" checks:

- Load config without UI and quit:
  - `nvim --headless "+qa"`
- Load config and run health checks:
  - `nvim --headless "+checkhealth" "+qa"`
- Sync/update plugins (networked):
  - `nvim --headless "+Lazy! sync" "+qa"`

## Lint and Format Commands

No single canonical lint script exists yet. Preferred checks are:

- Format Lua files with Stylua (if installed):
  - `stylua nvim/**/*.lua`
- Check Lua formatting only (no writes):
  - `stylua --check nvim/**/*.lua`
- Optional static lint (only if repo later adds config):
  - `luacheck nvim`

In-editor formatting is configured through `conform.nvim` in `nvim/lua/config/plugins/conform.lua`.

Configured formatters include:

- Lua: `stylua`
- Python: `ruff_format`
- Rust: `rustfmt`
- C: `clang_format`
- SQL: `sql_formatter`

## Test Commands

Current state:

- No formal `tests/` directory or test runner is currently configured.
- Treat headless startup and health checks as smoke tests.

Smoke test commands:

- Config loads cleanly:
  - `nvim --headless "+qa"`
- LSP setup file executes:
  - `nvim --headless "+lua dofile(vim.fn.stdpath('config') .. '/after/plugin/lsp.lua')" "+qa"`

If you add tests, prefer `plenary.nvim` test harness for Lua plugin configs.

Suggested test convention (for new tests):

- Place tests under `nvim/tests/`.
- Run all tests:
  - `nvim --headless -c "PlenaryBustedDirectory nvim/tests { minimal_init = 'nvim/tests/minimal_init.lua' }" -c qa`
- Run a single test file:
  - `nvim --headless -c "PlenaryBustedFile nvim/tests/path/to/file_spec.lua" -c qa`
- Run a single test by pattern (inside one file):
  - `nvim --headless -c "PlenaryBustedFile nvim/tests/path/to/file_spec.lua { filter = 'my test case name' }" -c qa`

## Code Organization Conventions

- `nvim/init.lua`: global options, bootstrap, high-level wiring.
- `nvim/lua/config/lazy.lua`: plugin manager bootstrap and plugin imports.
- `nvim/lua/config/plugins/*.lua`: one plugin spec per file, each returns a table.
- `nvim/lua/core/*.lua`: core behavior (keymaps, shared setup).
- `nvim/lsp/*.lua`: language server definitions/config tables.
- `nvim/after/plugin/*.lua`: post-load integration and runtime hooks.
- `nvim/after/ftplugin/*.lua`: filetype-specific settings.

## Style Guidelines (Lua)

Follow existing file-local style first; do not reformat unrelated lines.

### Formatting

- Prefer Stylua-compatible formatting.
- Preserve existing indentation style in touched files (tabs vs spaces differ today).
- Keep trailing commas in multiline tables.
- Avoid overly compact one-liners for non-trivial tables/functions.

### Imports and Requires

- Use local requires for reused modules, e.g. `local keymap = vim.keymap`.
- Use direct `require("module")` inline when only used once.
- Keep require strings stable and explicit; avoid dynamic require patterns.

### Types and Annotations

- Use EmmyLua annotations when helpful (`---@param`, `---@return`, `---@type`, `---@class`).
- Match existing annotation patterns in `nvim/lsp/*.lua`.
- Add types for non-obvious callback signatures and structured tables.

### Naming

- Local functions/variables: `snake_case`.
- Module file names: lowercase with hyphen or underscore following current folder style.
- User commands: `Lsp*`/descriptive PascalCase style as currently used.
- Keymap descriptions (`desc`) should be concise and action-first.

### Tables and Plugin Specs

- Plugin files should return a single spec table.
- Keep `opts`, `config`, `dependencies`, `event`, `keys` fields grouped logically.
- Prefer declarative `opts` where plugin supports it; use `config` for imperative setup.

### Error Handling and Notifications

- Prefer `vim.notify(...)` for recoverable/runtime user-facing issues.
- Use `pcall(...)` around optional integrations or fragile runtime calls.
- Reserve `error(...)` for truly exceptional conditions that should stop execution.
- In async callbacks, avoid throwing unless failure is unrecoverable.

### Keymaps

- Use `vim.keymap.set`.
- Always include `desc` for discoverability.
- Prefer buffer-local mappings when behavior is LSP/buffer-specific.
- Avoid mapping collisions; check existing `<leader>` mappings first.

### LSP and Diagnostics

- Register shared capabilities via centralized LSP setup (`after/plugin/lsp.lua`).
- Add server-specific logic in `nvim/lsp/<server>.lua`.
- Keep root detection deterministic and fast.
- Use `vim.diagnostic` APIs for diagnostics display/control.

### Comments and Docs

- Keep comments meaningful; avoid narrating obvious Lua syntax.
- Keep long rationale comments only when behavior is non-obvious.
- Update docs when adding new top-level commands or structural patterns.

## Change Management for Agents

- Make minimal, surgical edits.
- Do not silently rewrite unrelated formatting across files.
- When introducing a new dependency/plugin, place it in the appropriate plugin spec file.
- When adding new commands, document them in this file if they affect build/lint/test workflow.

## Quick Pre-PR Checklist

- Formatting check passed for touched Lua files.
- Headless Neovim startup works.
- `checkhealth` reviewed for regressions introduced by your change.
- Any new operational command documented in `AGENTS.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kovaxs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kovaxs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
