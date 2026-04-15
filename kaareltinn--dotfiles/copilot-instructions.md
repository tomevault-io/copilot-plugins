## dotfiles

> - Run test file: `<leader>t` (nvim)

# CLAUDE.md - Development Guidelines

## Test Commands
- Run test file: `<leader>t` (nvim)
- Run nearest test: `<leader>s` (nvim)
- Run last test: `<leader>l` (nvim)
- Run test suite: `<leader>a` (nvim)
- Ruby: `bundle exec rspec [file]`
- Elixir: `mix test [file:line]`
- Python: `pytest tests/path/to/test_file.py`

## Code Conventions
- Indentation: 2 spaces (tabstop=2, expandtab=true)
- Whitespace: No trailing spaces, tabs shown as '» '
- Format on save: Enabled for Elixir (mix), Go (goimports)
- Manual formatting: `<leader>fc` (LSP format)

## File Structure Patterns
- Ruby: `app/path/to/file.rb` ↔ `spec/path/to/file_spec.rb`
- Elixir: `lib/path/to/file.ex` ↔ `test/path/to/file_test.exs`
- TypeScript: `src/path/to/file.ts` ↔ `src/path/to/file.test.js`
- React: `src/path/to/Component.jsx` ↔ `src/path/to/Component.test.jsx`
- Python: `app/path/to/file.py` ↔ `tests/path/to/test_file.py`

## LSP & Tools
- TypeScript: ts_ls
- Ruby: solargraph
- Elixir: elixirls
- Python: ruff (linting)
- Go: gopls, ray-x/go.nvim

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaareltinn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kaareltinn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
