## jido-runic

> - **Compile**: `mix compile`

# JidoRunic Agent Development Guide

## Commands
- **Compile**: `mix compile`
- **Test all**: `mix test`
- **Test single file**: `mix test test/path/to/test_file.exs`
- **Test with coverage**: `mix coveralls.html`
- **Quality check**: `mix quality` (format, compile, credo, dialyzer, doctor)
- **Format code**: `mix format`
- **Type check**: `mix dialyzer`
- **Lint**: `mix credo`

## Code Style
- Elixir `~> 1.18` baseline
- Use `snake_case` for functions/variables, `PascalCase` for modules
- Add `@moduledoc` and `@doc` to all public functions
- Use `@spec` for type specifications
- Return tagged tuples: `{:ok, result}` or `{:error, reason}`
- Use Zoi schemas for struct validation
- Use Splode for error handling via `Jido.Runic.Error`
- Prefix test modules with namespace: `Jido.RunicTest.ModuleName`

## Architecture
- `Jido.Runic` - Main module, public API
- `Jido.Runic.Error` - Centralized Splode error handling

## Git Commit Guidelines
- Use conventional commit format: `type(scope): description`
- Types: feat, fix, docs, style, refactor, test, chore, ci

---
> Source: [agentjido/jido_runic](https://github.com/agentjido/jido_runic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
