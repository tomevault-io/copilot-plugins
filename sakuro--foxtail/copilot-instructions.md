## foxtail

> A Ruby implementation of [Project Fluent](https://projectfluent.org/) - a modern localization system designed to improve how software is translated.

# Foxtail - Code Instructions for AI Coding Agents

## What is Foxtail?

A Ruby implementation of [Project Fluent](https://projectfluent.org/) - a modern localization system designed to improve how software is translated.

## Documentation Map

| Purpose | Document |
|---------|----------|
| Project overview & usage | [README.md](README.md) |
| Architecture & design | [foxtail-runtime/doc/architecture.md](foxtail-runtime/doc/architecture.md) / [foxtail-tools/doc/architecture.md](foxtail-tools/doc/architecture.md) |
| Bundle system details | [foxtail-runtime/doc/bundle-system.md](foxtail-runtime/doc/bundle-system.md) |
| Custom functions | [foxtail-runtime/doc/custom-functions.md](foxtail-runtime/doc/custom-functions.md) |
| Language fallback | [foxtail-runtime/doc/sequence.md](foxtail-runtime/doc/sequence.md) |
| `icu4x` integration | [foxtail-runtime/doc/icu4x-integration.md](foxtail-runtime/doc/icu4x-integration.md) |
| CLI reference | [foxtail-tools/doc/cli.md](foxtail-tools/doc/cli.md) |

## Core Principles

### Language Policy

- **Code & documentation**: English
- **Commit messages**: English with `:emoji:` notation (e.g., `:sparkles:`, `:bug:`)
- **Chat**: Use the user's language

### Terminology

- **ICU4X**: Unicode org's internationalization project
- **`icu4x`**: Ruby gem providing ICU4X bindings

### Skills

- Explore available skills and use them proactively when applicable

### Memory

- Propose updating MEMORY.md at session boundaries

## Development Commands

```bash
bundle exec rake            # Run all checks (spec + rubocop)
bundle exec rspec           # Run tests
bundle exec rubocop -a      # Auto-fix style
bin/console                 # Interactive console
bundle exec rake doc        # Generate YARD documentation
```

**Note**: Run commands from within each subproject directory (`foxtail-runtime/` or `foxtail-tools/`). Running `rspec` from the repository root finds no tests because specs are located in subproject directories.

## Configuration

- **Ruby version**: >= 3.3
- **RuboCop style**: Double quotes for strings
- **Release**: Automated via CI workflow

See [foxtail-runtime/doc/architecture.md](foxtail-runtime/doc/architecture.md) and [foxtail-tools/doc/architecture.md](foxtail-tools/doc/architecture.md) for detailed architecture information.

---
> Source: [sakuro/foxtail](https://github.com/sakuro/foxtail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
