## nvim-dora

> - Run the full smoke suite with `./scripts/smoke.sh`. The tests live in

# Repository Notes

- Run the full smoke suite with `./scripts/smoke.sh`. The tests live in
  `scripts/tests/*.lua`, one file per area, run in order by
  `scripts/smoke.lua`; run a single file with
  `DORA_TEST_FILE=scripts/tests/<name>.lua` using the nvim invocation from
  `scripts/smoke.sh` (later files may assume state left by earlier ones, so
  the full suite is authoritative).
- When running the smoke suite from Codex, use sandbox escalation. Neovim's
  filesystem watchers may repeatedly fire inside the sandbox and stall the run.
- Regenerate README config docs with `PANVIMDOC_DIR=~/code/panvimdoc ./scripts/docs.sh`.
- For ad hoc headless Neovim checks, set `NVIM_LOG_FILE=/dev/null` to avoid creating `nvim.log` in the repo and use `set noswapfile` to prevent swap-file errors.
- When running `lua-language-server --check` directly, set `VIMRUNTIME` from
  Neovim first. Otherwise `$VIMRUNTIME/lua` in `.luarc.json` does not resolve
  and LuaLS reports spurious undefined Neovim/LSP types. For example:
  ```sh
  VIMRUNTIME="$(NVIM_LOG_FILE=/dev/null nvim --headless -u NONE -i NONE --noplugin \
    -c 'set noswapfile' -c 'lua io.stdout:write(vim.env.VIMRUNTIME)' -c qa)" \
    lua-language-server --check=. --checklevel=Hint --configpath=.luarc.json \
    --logpath="${TMPDIR:-/tmp}/dora-luals-log" \
    --metapath="${TMPDIR:-/tmp}/dora-luals-meta"
  ```

## Architecture

Each dora window is a session: a scratch buffer named after the browsed
directory, with a `DoraState` (cwd, cached listings, filter, marks) registered
in `dora/store.lua`. Actions mutate that state or the filesystem, then
re-render. The modules:

- `plugin/dora.lua` — `:Dora`, highlight defaults, and the directory auto-open
  autocmd. Requires no dora module until a command or autocmd fires.
- `lua/dora.lua` — the config table and `configure()`. Modules capture
  `require'dora'.config` at require time, so the table is merged in place and
  never reassigned.
- `lua/dora/api.lua` — every user-facing action (`M.*`), addressable by name
  from the keymap config. Also owns session-global state: expanded
  directories, cut/copy marks, trash history.
- `lua/dora/actions.lua` — metadata registry for built-in actions
  (description, help section, visual variant); keymaps.lua and help_win.lua
  derive their tables from it.
- `lua/dora/view.lua` — the render pipeline: listing cache (with fs watchers)
  → tree/filter rows → buffer text + extmarks; also cursor placement.
- `lua/dora/tree.lua` — operations on the expanded-directories set (folding,
  and following renames/restores).
- `lua/dora/store.lua` — dora buffer → `DoraState` registry.
- `lua/dora/fs.lua` — filesystem ops, sync and async; renames editor buffers
  so they follow moved files.
- `lua/dora/lsp.lua` — native LSP `willRenameFiles`/`didRenameFiles` handling,
  including server registration filters.
- `lua/dora/buffer.lua` — dora buffer creation/naming and the
  buffer-follows-file renames.
- `lua/dora/keymaps.lua` — installs keymaps and prefix-hint windows; requires
  api.lua lazily to avoid a require cycle.
- `lua/dora/ui/{confirm,filter,help,info,preview,prompt}.lua` — UI windows,
  sharing `lua/dora/ui/window.lua` for float layout.

## Adding an action

1. Implement `M.<name>()` in `lua/dora/api.lua` (plus a `<name>_visual`
   variant if it should act on visual selections).
2. Register it in `lua/dora/actions.lua` with a description, help section,
   and (if any) its visual variant.
3. To ship a default mapping, add it to the keymaps in `lua/dora.lua` and
   regenerate docs with `./scripts/docs.sh`.
4. Cover it in the smoke suite (`scripts/tests/`).

---
> Source: [beardedsakimonkey/nvim-dora](https://github.com/beardedsakimonkey/nvim-dora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
