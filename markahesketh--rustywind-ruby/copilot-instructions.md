## rustywind-ruby

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This gem wraps the [rustywind](https://github.com/avencera/rustywind/) CLI binary (a Tailwind CSS class sorter) for use in Ruby projects. It downloads and packages platform-specific binaries for macOS, Linux, and Windows.

## Common Commands

- `bin/setup` - Install dependencies
- `rake test` - Run tests
- `bundle exec standardrb` - Run linter (Standard Ruby)
- `bundle exec standardrb --fix` - Auto-fix lint issues
- `bundle exec rustywind` - Run the wrapped binary

## Architecture

The gem has two version constants that must be kept in sync when updating:
- `lib/rustywind/ruby/version.rb` - Gem version (`Rustywind::Ruby::VERSION`)
- `lib/rustywind/ruby/upstream.rb` - Upstream rustywind version (`Rustywind::Ruby::Upstream::VERSION`)

Key components:
- `exe/rustywind` - Entry point that executes the platform-appropriate binary
- `lib/rustywind/ruby/binary.rb` - Detects platform/arch and returns the correct binary path
- `lib/rustywind/ruby/upstream.rb` - Maps platform/arch to upstream binary URLs

Binaries are stored in `exe/{platform}/{arch}/rustywind` and downloaded via `rake download_binaries`.

## Release Process

1. Update version in both `version.rb` and `upstream.rb`
2. `bundle exec rake clobber` - Remove old binaries
3. `bundle exec rake download_binaries` - Download new binaries
4. Commit changes and update CHANGELOG.md
5. `bundle exec rake release` - Tag and push to RubyGems

---
> Source: [markahesketh/rustywind-ruby](https://github.com/markahesketh/rustywind-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
