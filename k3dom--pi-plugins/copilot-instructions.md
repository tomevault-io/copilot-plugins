## pi-plugins

> This repository is a pnpm/Turbo monorepo containing plugins/extensions for the

# pi-plugins

This repository is a pnpm/Turbo monorepo containing plugins/extensions for the
pi-agent harness. Pi-agent plugins live under `plugins/`; shared tooling and support
packages live under `tooling/`.

## Validation

Use the `ci` command to validate code changes (format, lint, type-check, build).

## Vendored Repositories

This project vendors external repositories of key libraries as git subtrees under
`.agents/repos/`. Use vendored repositories as read-only reference material when
working with related libraries to explore API's, find usage examples, and understand
implementation details.

- Effect-TS v4 (`.agents/repos/effect/packages/`)
- pi-agent (`.agents/repos/pi/`)

Subtrees are tracked in `.agents/repos/.subtrees`, pinned to the release each
dependency is on, and managed with `scripts/add-subtree.sh` (add a new repo) and
`scripts/update-subtree.sh` (pull the registered ref).

## Agent skills

### Issue tracker

Issues and PRDs live in GitHub Issues on `k3dom/pi-plugins`, via the `gh` CLI. See
`docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles, each label string equal to its name. See
`docs/agents/triage-labels.md`.

### Domain docs

Multi-context: a root `CONTEXT-MAP.md` pointing at per-package `CONTEXT.md` files.
See `docs/agents/domain.md`.

---
> Source: [k3dom/pi-plugins](https://github.com/k3dom/pi-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
