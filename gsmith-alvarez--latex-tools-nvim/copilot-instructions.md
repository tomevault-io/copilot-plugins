## latex-tools-nvim

> Orientation for AI agents picking up work on this plugin cold. Read this before editing.

# AGENTS.md — latex-tools.nvim

Orientation for AI agents picking up work on this plugin cold. Read this before editing.

## What this plugin is

Neovim plugin providing LuaSnip-powered LaTeX math snippets, visual wrappers, and matrix editing for markdown (`$...$`, `$$...$$`) and `.tex` files. Extracted from the user's dotfiles at `~/dotfiles/nvim/lua/snippets/` into a standalone plugin.

User-facing docs live in `README.md`. This file is for *contributors* (you).

## Layout

```
lua/latex-tools/
  init.lua            -- setup() entry + default config + submodule exports
  context.lua         -- math zone / matrix env / code block detection (self-contained)
  math_parser.lua     -- tokenizer for smart-fraction / smart-postfix (verbatim copy, no deps)
  snippets.lua        -- ALL built-in snippets; register(config) called from setup()
  matrix.lua          -- handle_enter() for matrix row separator; consumed by autolist.lua
  visual_wrappers.lua -- <leader>nu/no/nc/nk/nb visual-mode wrappers
plugin/latex-tools.lua  -- intentionally empty; setup() must be called explicitly
tests/
  minimal_init.lua    -- adds plugin to rtp, loads mini.nvim from dotfiles config
  test_context.lua    -- math zone + matrix env + code block tests (MiniTest)
  test_math_parser.lua
Makefile              -- `make test` runs all tests headlessly
```

## How it's wired into the dotfiles

- `~/dotfiles/nvim/lua/core/pack.lua` declares the plugin via `gh 'gsmith-alvarez/latex-tools.nvim'` in the deferred section, with `'latex-tools.nvim'` in `M.deferred_names`. This is identical to `sigils.nvim` / `golem-be-good.nvim` — `vim.pack` handles install/update and `packadd` handles deferred loading.
- `~/dotfiles/nvim/lua/plugins/editing/luasnip.lua` calls `require('latex-tools').setup()` inside the LuaSnip config block (no manual `rtp` manipulation needed).
- `~/dotfiles/nvim/lua/core/autolist.lua` does `require('latex-tools.matrix')` and calls `matrix_shortcuts.handle_enter()` on `<CR>`.
- `~/dotfiles/nvim/lua/snippets/markdown/math.lua` is a stub returning `{}, {}` — all math snippets live here now.
- `~/dotfiles/nvim/lua/snippets/markdown.lua` keeps only text/callout snippets (anything with `is_plain_text` condition).

Do NOT re-add math snippets to the dotfiles. They belong here.

## Running tests

```bash
cd ~/code/vibe/nvim-plugins/latex-tools.nvim
make test                                    # all tests
make test-file FILE=tests/test_context.lua   # single file
```

Tests run headlessly via `nvim --headless --noplugin -u tests/minimal_init.lua`. `minimal_init.lua` pulls `mini.nvim` from `~/.config/nvim/pack/core/opt/mini.nvim` — if that path moves, tests break.

## Critical design points (do not regress these)

### 1. `context.lua` is self-contained

It MUST NOT `require('snippets.utils')` or anything from the dotfiles. The only exception is `is_in_code_block()` which does a defensive `pcall(require, 'snippets.utils')` for a possible affirmative result and ALWAYS falls through to its own scanner. Keep it that way — users of this plugin won't have the dotfiles.

### 2. Markdown math zone detection: fallback is authoritative

`_check_mathzone_markdown()` only trusts Treesitter's **affirmative** (`true`) result. On `false` or `nil` it always runs the line-scanner fallback. Reason: the markdown-LaTeX injection tree often sits under a `text_mode` node, and generic-command heuristics (`\text`, `\intertext`) can match unrelated injected text, producing spurious negatives.

In `.tex`/`.latex` files, `text_mode` IS a real `\text{}` escape, so we return `false` there. This filetype split is load-bearing — don't collapse it.

### 3. Fallback scanner truncates to `col`, not `col + 1`

`_check_mathzone_fallback()` does `lines[#lines] = lines[#lines]:sub(1, col)`. This excludes the character AT the cursor. In insert mode at `$int|$` (cursor on the closing `$`), col=4 and the scanner must see only `$int` — including the closing `$` would toggle `inline_math` back to false.

There's a regression test: `'is_in_mathzone returns true at closing $ (insert mode position)'`. If you change this, that test will tell you.

### 4. Snippets registered programmatically, not via `from_lua` loader

`snippets.lua` calls `ls.add_snippets('markdown', ...)` and `ls.add_snippets('tex', ...)` directly. This is intentional — the plugin has no snippet dir for LuaSnip's `from_lua` loader to find. If you're tempted to add a `luasnippets/` directory, don't; keep everything in `snippets.lua` so `overrides`/`disable`/`extra` plumbing stays in one place.

### 5. `overrides` / `disable` / `extra` pipeline

`snippets.register(config)` builds the snippet list, then:
- applies `config.snippets.overrides` via the `sa()` wrapper (patches `trig`/`wordTrig`/`regTrig` at construction time)
- filters out anything in `config.snippets.disable`
- appends `config.snippets.extra` AFTER built-ins (not subject to overrides/disable)

Overrides are keyed by the ORIGINAL trigger string — for regex snippets that's the raw Lua pattern. Body/nodes are not patchable; only the three trigger fields.

### 6. Snippets register once per filetype

Registration loops over `{'markdown', 'tex'}`. Do not add a third call for the same snippets — earlier there was a bug where snippets registered 4× causing ordering issues. Commit `951b38b` fixed it.

## Where to look for specific behaviors

| Question | File |
|---|---|
| "Why doesn't my snippet fire in `$...$`?" | `context.lua:_check_mathzone_fallback` + `_check_mathzone_markdown` |
| "How does smart-fraction pick the numerator?" | `math_parser.lua` (tokenizer) + the `/` snippet in `snippets.lua` |
| "Why is `Enter` inserting ` \\ `?" | `matrix.lua:handle_enter` called from `autolist.lua` |
| "How does `,,` become ` & `?" | `snippets.lua` matrix column snippet (condition: `in_matrix_env`) |
| "Where's the default keymap for `<leader>nu`?" | `visual_wrappers.lua:setup` |

## Debugging math zone false negatives

1. Open a markdown buffer with `$...$` content, put cursor inside.
2. `:lua print(require('latex-tools.context').is_in_mathzone())` — should print `true`.
3. If false: check filetype (`:set ft?`), then bypass TS with `:lua print(require('latex-tools.context')._check_mathzone_fallback())`.
4. If fallback says true but public API says false, the TS path is returning `false` — check if a new node type needs to be added to `TS_MATH_NODES` or if the `text_mode` / generic-command heuristic is over-matching.

## Git / commit style

Conventional commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`. Keep commits small and focused — this repo has a history of single-concern commits and reviewers (and future debugging) depend on that.

## Things NOT to do

- Don't add dotfiles dependencies (`snippets.utils`, `snippets.math_parser`, etc.) except the defensive pcall in `is_in_code_block`.
- Don't replace MiniTest with another framework — `minimal_init.lua` is pinned to it.
- Don't touch `col` arithmetic in `_check_mathzone_fallback` without re-running the closing-`$` regression test.
- Don't collapse the markdown/tex branches in `_check_mathzone_treesitter` — they behave differently on purpose.
- Don't auto-setup on plugin load. `setup()` is explicit so users control ordering relative to LuaSnip.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gsmith-alvarez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
