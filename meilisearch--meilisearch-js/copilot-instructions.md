## meilisearch-js

> Use `pnpm`, not `npm`.

# AGENTS.md

Use `pnpm`, not `npm`.

This repository provides a `docker-compose.yml` that lets you develop and run tests without installing Meilisearch or Node.js locally.

## Commmands

Run the commands inside Docker using `docker compose run --rm package bash -c "<command>"`.

- `pnpm test` - run tests
- `pnpm test path/to/file.test.ts` - run specific test files
- `pnpm style:fix` - lint and format the code

## Workflow

- Lint and format code after finishing a task

---
> Source: [meilisearch/meilisearch-js](https://github.com/meilisearch/meilisearch-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
