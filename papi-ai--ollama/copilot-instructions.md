## ollama

> Papi is the best standalone AI agent library in PHP. Framework-agnostic, minimal dependencies, type-safe.

# Papi Provider — Development Guidelines

## Project Vision

Papi is the best standalone AI agent library in PHP. Framework-agnostic, minimal dependencies, type-safe.

## Quick Reference

```bash
composer lint          # Check code style (PHP CS Fixer, PSR-12)
composer lint:fix      # Auto-fix code style
composer analyse       # Static analysis (Psalm level 4)
composer test          # Run tests (Pest)
composer test:coverage # Run tests with 75% minimum coverage
composer ci            # Run all checks (lint + analyse + test:coverage)
```

## Code Standards

- **PHP 8.2+** with `declare(strict_types=1)` in every file
- **PSR-12** coding style, enforced by PHP CS Fixer
- **Psalm level 4** static analysis must pass with zero errors
- **75% minimum test coverage**, enforced in CI and pre-commit hook
- **Pest** for testing with describe/it syntax

## Architecture Rules

- **ext-curl only** — direct HTTP via cURL, no Guzzle or HTTP abstraction layers
- **Provider packages are thin** — one main class converting between core types and API formats
- **Implement core interfaces** — ProviderInterface and any capability interfaces from papi-core
- **Immutable value objects** — use core Message, Response, ToolCall types
- **Format conversion** — each provider converts core types to its API format and back

## Testing

- Use Pest's `describe()` / `it()` syntax
- Mock HTTP responses, don't call real APIs
- Test message conversion, tool format conversion, streaming parsing
- Every public method needs test coverage

## Git Workflow

- Install pre-commit hook: `cp .hooks/pre-commit .git/hooks/pre-commit`
- All checks must pass before committing
- CI runs lint, static analysis, and tests across PHP 8.2–8.5

---
> Source: [papi-ai/ollama](https://github.com/papi-ai/ollama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
