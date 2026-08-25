## elasticsearch-php

> Always use `composer` commands, not any other package manager.

# AGENTS.md

## Setup commands

Always use `composer` commands, not any other package manager.

- Install dependencies: `composer install`
- Run unit tests: `composer run-script test`
- Run static analysis: `composer run-script phpstan`
- Run integration tests (requires Elasticsearch running): `ELASTICSEARCH_URL="http://elastic:${ES_LOCAL_PASSWORD}@localhost:9200" TEST_SUITE="free" composer run-script integration-test`

## Testing

**The entire unit test suite (`composer run-script test`) must pass and exit cleanly before you commit code.**

Integration tests require a running Elasticsearch instance. Skip them if one is not available.

## Project Structure

- **src/** - PHP source files
- **tests/** - Unit and integration tests
- **docs/** - Documentation

## Development Workflow

1. Make changes to `src/` files
2. Run `composer run-script phpstan` to check for static analysis errors
3. Run `composer run-script test` to verify all unit tests pass

No build step is needed - PHP is interpreted directly.

## OS Compatibility

All code in `src/` and `tests/`, and any scripts run by `composer`, must work on Linux, macOS, and Windows.

## Adding new agent instructions

If a specific action you learned to do better will be useful to other agents doing the same task in the future, but may not be needed for all agent-related tasks, create or update skills in `.github/skills/`.

If you learned something that will be useful to any contributor to this project, update `AGENTS.md`.

---
> Source: [elastic/elasticsearch-php](https://github.com/elastic/elasticsearch-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
