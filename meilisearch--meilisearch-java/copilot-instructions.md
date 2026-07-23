## meilisearch-java

> This repository provides a `docker-compose.yml` that lets you develop and run tests without installing Java locally.

# AGENTS.md

This repository provides a `docker-compose.yml` that lets you develop and run tests without installing Java locally.

To speed up local development, always use Docker Compose with the `-v gradle-cache:/root/.gradle` option to **persist the Gradle cache across Docker runs**.

## Commands

- `docker compose run --rm package bash -c "bash ./scripts/lint.sh"` - lint code
- `docker compose run --rm package ./gradlew test` - run unit tests
- `docker compose run --rm package ./gradlew test integrationTest` - run integration tests

## Workflow

- Lint code after changes
- Run tests after changes

---
> Source: [meilisearch/meilisearch-java](https://github.com/meilisearch/meilisearch-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
