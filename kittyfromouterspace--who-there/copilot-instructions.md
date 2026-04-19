## who-there

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Elixir project called `WhoThere` created from a template. It's a minimal Elixir library project with standard structure.

## Common Commands

### Development
- `mix compile` - Compile the project
- `mix test` - Run all tests
- `mix test test/specific_test.exs` - Run a specific test file
- `mix format` - Format code according to project standards
- `mix format --check-formatted` - Check if code is properly formatted

### Documentation
- `mix docs` - Generate documentation (if ExDoc is added as dependency)
- `iex -S mix` - Start interactive Elixir shell with project loaded

### Dependencies
- `mix deps.get` - Fetch dependencies
- `mix deps.update --all` - Update all dependencies

## Project Structure

```
lib/
  who_there.ex          # Main module with basic hello/0 function
test/
  who_there_test.exs    # Tests for main module
  test_helper.exs       # Test configuration
mix.exs                 # Project configuration and dependencies
.formatter.exs          # Code formatting configuration
```

## Architecture Notes

- Standard Elixir library structure
- Currently contains only a basic "hello world" module
- Uses ExUnit for testing with doctests enabled
- No external dependencies configured yet
- Targets Elixir ~> 1.18

## Code Style

- Uses standard Elixir formatting via `mix format`
- Formatter configured for inputs: `["{mix,.formatter}.exs", "{config,lib,test}/**/*.{ex,exs}"]`
- Follow standard Elixir naming conventions (snake_case for functions, PascalCase for modules)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kittyfromouterspace) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
