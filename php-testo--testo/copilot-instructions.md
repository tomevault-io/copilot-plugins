## testo

> Instructions for AI coding agents (Claude Code, Codex, Aider, Cursor, …) working in a project that uses **Testo** for testing.

# AGENTS.md

Instructions for AI coding agents (Claude Code, Codex, Aider, Cursor, …) working in a project that uses **Testo** for testing.

If you are contributing **to Testo itself**, ignore this file and read `CLAUDE.md` in the Testo repository instead.

## What Testo is

**Testo** is an extensible PHP testing framework (PHP 8.1+) for projects that need substantial customization of testing workflows — SDKs, framework tooling, complex integrations. It uses familiar PHP syntax (no DSL) with attribute-based test configuration and a middleware system for deep customization.

Core surface you will encounter:

- Attributes: `#[Test]`, `#[RetryPolicy]`, `#[ExpectException]`, and others.
- No base class requirement for test classes.
- Built-in DI container; assertions via a fluent `Assert` facade.
- Symfony Console-based CLI (`vendor/bin/testo`).
- Configuration lives in a `testo.php` file at the project root.

## Required reading before writing or modifying Testo tests

Testo publishes machine-friendly docs in the `llms.txt` format. **Fetch the appropriate one before writing tests, middleware, or extensions** — they describe the public API faithfully, which is hard to reconstruct from source alone.

- **<https://php-testo.github.io/llms.txt>** — concise index of Testo's public API: attributes, assertions, configuration entry points. Use this for routine test-writing tasks.
- **<https://php-testo.github.io/llms-full.txt>** — full expanded documentation: middleware architecture, plugin authoring, dependency injection, console commands, lifecycle hooks, etc. Use this when extending Testo or doing non-trivial customization.

Rule of thumb: start with `llms.txt`; escalate to `llms-full.txt` when the short index does not answer the question. Prefer these sources over guessing from class names or older PHPUnit knowledge — Testo is **not** PHPUnit and the APIs differ.

## Pitfalls to avoid

- **Do not assume PHPUnit semantics.** Testo's `Assert` facade, lifecycle, data providers, and exception expectations are its own. Verify against `llms.txt` before writing.
- **Do not mock enums or `final` classes** — use real instances.
- **Do not invent attributes.** If you need behavior that is not in `llms.txt`, look in `llms-full.txt` before inventing an attribute or middleware that does not exist.
- **Run tests via the Testo CLI** (`vendor/bin/testo`), not via `phpunit`.

---
> Source: [php-testo/testo](https://github.com/php-testo/testo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
