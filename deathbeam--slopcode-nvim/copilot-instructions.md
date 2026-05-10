## slopcode-nvim

> Neovim plugin: streaming AI chat with tool-use via OpenAI-compatible `/chat/completions` and `/responses` endpoints.

# AGENTS.md — slopcode

Neovim plugin: streaming AI chat with tool-use via OpenAI-compatible `/chat/completions` and `/responses` endpoints.

## Commands

```sh
make lint          # check formatting with stylua (--check mode)
make test          # run full test suite (custom mini.test runner, headless Neovim)
make format        # format all Lua files with stylua
```

## Testing

Tests use [mini.test](https://github.com/nvim-mini/mini.test) with busted-style `describe`/`it`/`before_each`. Dependencies (mini.test, async.nvim) auto-clone into `.deps/` on first run.

- **All tests use child Neovim**: `local child = MiniTest.new_child_neovim()` then `child.restart({ "-u", "scripts/init.lua" })` in `before_each`. The `-u` flag is required — it sets up runtimepath + deps.
- Assertions: `MiniTest.expect.equality(a, b)` / `MiniTest.expect.no_equality(a, b)`

## Critical Architecture Rules

**Only `loop.lua` touches the buffer.** All buffer writes flow through `loop.push()` → `loop.drain()` (16ms batch timer). Never call `nvim_buf_set_lines`, `append`, or any buffer API from agent, tools, callbacks, or streaming handlers.

**All I/O is async** via [async.nvim](https://github.com/nvim-lua/async.nvim) coroutines. If you're in a fast event (check `vim.in_fast_event()`), call `sync()` (from `utils.vim` or `tools.lua`) to yield before any vim.fn/vim.api calls. Blocking APIs will freeze Neovim.

**Singleton modules, not instances.** `agent.lua`, `loop.lua`, `status.lua` store state in module-level locals (prefixed `_`). No `M.new()` pattern. Lifecycle is `attach`/`detach`.

**Shared references use in-place mutation.** `agent.messages` is a reference to the internal `_messages` table. Reset and compact use `for k in pairs(t) do t[k] = nil end` — never reassign the table, or external references break.

**Notifications go through `status.notify()`**, `status.notify` renders in the winbar AND calls `vim.notify`

---
> Source: [deathbeam/slopcode.nvim](https://github.com/deathbeam/slopcode.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
