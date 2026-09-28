## beside-nvim

> beside.nvim renders the current document in a split beside it, with leaf, glow,

# AGENTS.md

beside.nvim renders the current document in a split beside it, with leaf, glow,
pandoc or any renderer added to `config.renderers`. Neovim 0.10+, no dependencies.

## Layout

- `lua/beside/init.lua`: public API in `Beside`, helpers in `H` under `-- Name ====` banners, runtime state in `H.state` only.
- `lua/beside/renderers.lua`: the renderer contract and the builtins.
- `lua/beside/ansi.lua`, `lua/beside/anchors.lua`: pure. SGR to text and spans; source line to rendered line matching.
- `lua/beside/health.lua`, `plugin/beside.lua`, `doc/beside.txt`: health, `:Beside`, hand-written help. The README mirrors the config and renderer sections; update both with the code.
- `test/run.lua`: headless unit checks, no renderer needed. `demo/`: vhs tapes for the README gifs, `make demo`.

## Verify

`make test` and `make format-check` (stylua 2.3.1). Editor behaviour is tested by hand with `demo/sample.*`.

## Conventions

- mini.nvim shape: one statement per line, full names, a one-line why-comment above a block, LuaCATS on public functions.
- Config deep-merges over the defaults and is type-checked in `H.setup_config`; lists replace. `vim.b.beside_config` layers per buffer.
- Renderers are data: `{ filetypes, command(context), env?, executable? }`. `prefer[ft]` orders them, the first installed wins.
- Sync is a heuristic: each source line's text is searched in the render, positions between matches interpolated. Each side records the view it set on the other so a scroll is not synced back.
- On purpose: documents only (no report generators), no diff-based updates, no render cache.

---
> Source: [mariocesar/beside.nvim](https://github.com/mariocesar/beside.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
