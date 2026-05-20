## phoenix-react

> - **Build**: `mix deps.get && npm install`

# Agent Guidelines for Phoenix.React

## Commands
- **Build**: `mix deps.get && npm install`
- **Format**: `mix format` (uses .formatter.exs config)
- **Test**: `mix test` (requires 3s delay for server startup in test_helper.exs)
- **Single test**: `mix test path/to/test.exs:line_number`
- **Bundle React components**: `mix phx.react.bun.bundle --component-base=assets/component --output=priv/react/server.js`

## Code Style
- **Elixir**: Follow standard Elixir conventions, use `mix format`
- **Module names**: PascalCase (e.g., `Phoenix.React.Server`)
- **Function names**: snake_case
- **Types**: Use `@type` and `@spec` for all public functions
- **Error handling**: Return `{:ok, result}` or `{:error, reason}` tuples
- **Imports**: Alias modules at top, use qualified calls when needed
- **GenServer**: Use `@impl true` for callback implementations
- **Documentation**: Include `@moduledoc` and `@doc` for public modules/functions
- **React components**: Export `Component` function, use JSX syntax in .js files

---
> Source: [gsmlg-dev/phoenix-react](https://github.com/gsmlg-dev/phoenix-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
