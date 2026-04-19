## array-proc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`array_proc` is a Ruby gem that adds a `to_proc` method to `Array`, enabling `&` shorthand to call multiple methods on each element of an enumerable (e.g., `[0, 5].map(&[:zero?, :even?])`).

## Commands

- **Install dependencies:** `bundle install`
- **Run all checks (tests + lint):** `bundle exec rake`
- **Run tests:** `bundle exec rspec`
- **Run a single test file:** `bundle exec rspec spec/array_proc_spec.rb`
- **Run a single test by line:** `bundle exec rspec spec/array_proc_spec.rb:LINE`
- **Lint:** `bundle exec rubocop`
- **Lint with autofix:** `bundle exec rubocop -A`
- **Coverage report:** `bundle exec rake coverage`

## Architecture

The entire gem is a single file (`lib/array_proc.rb`) that monkey-patches `Array#to_proc`. It returns a proc that maps each array element through `method.to_proc.call(object)`. Tests are in `spec/array_proc_spec.rb` using RSpec.

## Linting

RuboCop is configured (`.rubocop.yml`) with plugins: `rubocop-performance`, `rubocop-rake`, `rubocop-rspec`. All new cops are enabled by default.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agrberg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
