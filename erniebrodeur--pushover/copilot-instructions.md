## pushover

> - Prefer modern Ruby patterns over legacy implementation patterns.

# Project Conventions

## Design

- Prefer modern Ruby patterns over legacy implementation patterns.
- Preserve released public behavior when practical, but do not preserve obsolete implementation details unless they are explicitly part of the supported contract.

## Ruby runtime

- Run Ruby, Bundler, Rake, RSpec, RuboCop, and other Ruby commands through `/opt/homebrew/bin/rbenv exec`.
- The Codex shell may resolve `/usr/bin/ruby` (Apple Ruby 2.6) instead of the project's Ruby version.
- The project runtime is defined by `.ruby-version`.

## HTTP client

- Use Excon for Pushover HTTP requests. Do not introduce another HTTP client without a specific need.
- Use Excon's request stubs for mocked HTTP tests.
- Consult the installed Excon documentation or source before assuming library behavior.
- it has no context 7 documentation, so read the source code for details.

---
> Source: [erniebrodeur/pushover](https://github.com/erniebrodeur/pushover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
