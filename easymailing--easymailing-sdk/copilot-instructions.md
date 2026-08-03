## easymailing-sdk

> Monorepo of the official Easymailing backend SDKs (Node + PHP). This file

# easymailing-sdk Agent Guide

Monorepo of the official Easymailing backend SDKs (Node + PHP). This file
documents conventions any contributor — human or AI assistant — should
follow when working on this repository.

## Packages

- `packages/contract` — normalized OpenAPI + breaking-change detection. Internal, not published.
- `packages/node` — `@easymailing/sdk` for Node and TypeScript backends (npm).
- `packages/php` — `easymailing/sdk-php` for PHP backends (Packagist).

## Tooling

- **Node**: pnpm 9.x workspaces, Node 20+, ESM only.
- **PHP**: Composer, PHP 8.1+, PSR-4, named arguments, `public readonly` properties.

## Conventions

- All generated files must include the `AUTO-GENERATED ... DO NOT EDIT BY HAND` header.
- `git diff --exit-code` must be zero on generated paths in CI.
- `packages/contract/sdk-map.yml` is the shared language-neutral public SDK naming map.
- Generated SDK resource files under `packages/node/src/generated/resources/` and
  `packages/php/src/Generated/Resources/` must not be edited by hand.
- Generated SDK resource references under `packages/node/docs/resources.md` and
  `packages/php/docs/resources.md` must not be edited by hand.
- Deprecated OpenAPI operations must not be exposed in SDK resources.
- Commit messages follow Conventional Commits (used by changesets and release-please).
- Source of truth for SDK shape is `packages/contract/openapi.json` — never hand-edit it.
- Write commit messages and public docs in English. Code comments may be in either language.

## Releases

- Node uses [changesets](https://github.com/changesets/changesets). Add a changeset with `pnpm changeset`. Merging to `main` triggers a release PR; merging the release PR publishes to npm.
- PHP uses [release-please](https://github.com/googleapis/release-please-action). Merging to `main` triggers a release PR; merging the release PR creates a Git tag, which Packagist auto-detects.
- Both packages are currently in `alpha` prerelease. To exit prerelease, run `pnpm changeset pre exit` and update `prerelease-type` in `.release-please-config.json`.

## Safety

Never publish to npm or Packagist, never push tags, never run release workflows
manually without explicit human approval. Same applies to destructive git
commands (`push --force`, `reset --hard`, etc.).

---
> Source: [easymailing/easymailing-sdk](https://github.com/easymailing/easymailing-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
