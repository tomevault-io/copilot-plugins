## python-agentic-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Local Dev

Use the `justfile` recipes for local development.

Always run `just check` to ensure the code meets code quality standards.

## Architecture

Follow domain driven design to organise the code.

```
src/
├── domain/                    # Core business logic (pure, no dependencies)
├── services/                  # Application services (orchestration)
├── adapters/                  # Adapters to 3rd party systems and tools
│   └── repository/            # Database implementations
├── entrypoints/               # External entrypoints (HTTP, NATS event handlers etc.)
│   └── routes/                # HTTP route handlers
├── infra/                     # Infrastructure concerns
│   └── database/              # Database connection, migrations, queries
└── config.py                  # Configuration for the application
└── app.py                     # the server application
```

## Test Driven Development

When working on new functionality, first write BDD style GIVEN WHEN THEN tests that test behaviour and not implementation detail. With the test in place, then write the code so the test passes.  

Strive for 100% test coverage and ensuring all tests pass before marking the task as complete.

Tests are located under `tests/` and never under `src/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sam-atkins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
