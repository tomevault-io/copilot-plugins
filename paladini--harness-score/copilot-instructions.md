## harness-score

> A monorepo shipping four artifacts about harness engineering for Cursor:

# Agent Guide — harness-score

## What this is

A monorepo shipping four artifacts about harness engineering for Cursor:
the guide (VitePress → GitHub Pages), the deterministic `harness-score` CLI,
the Cursor Marketplace plugin, and a GitHub Action. The repo dogfoods its own
scanner: it must always score **L4** (`npm run scan`).

## Layout

- `packages/cli/` — the scanner. TypeScript, ESM, **zero runtime deps**.
  - `src/checks/` — one file per dimension (see `.cursor/rules/checks.mdc`)
  - `src/score.ts` — the maturity rubric (levels L0–L4)
- `docs/` — the VitePress guide. `docs/guide/measure-and-improve.md` holds
  the check catalog with one `{#<check-id>}` anchor per check.
- `plugin/` — Cursor plugin (`.cursor-plugin/plugin.json`, command, skill).
- `action/` — composite GitHub Action wrapping the CLI.
- `fixtures/level-0..4/` — sample repos pinned to each maturity level by
  tests. Changing a check usually changes a fixture.

## Build & test

- `npm test` — builds the CLI and runs vitest (includes docs-sync tests).
- `npm run lint` — Biome (lints + checks formatting).
- `npm run scan` — self-audit; must report L4.
- `npm run docs:build` — builds the guide; must pass (dead links fail it).
- Tests MUST pass before any commit.

## Non-negotiable conventions

- The CLI stays 100% deterministic: no LLM calls, no network, no telemetry,
  no `Date.now()`-dependent output. Filesystem reads and parsing only.
- `packages/cli` keeps **zero runtime dependencies** (fast `npx`, no supply
  chain surface). Dev dependencies are fine.
- The rubric lives in three places that must change together:
  `src/score.ts` (implementation), `docs/guide/maturity-model.md` (levels),
  `docs/guide/measure-and-improve.md` (check catalog). The docs-sync test
  enforces anchors and point values.
- Check IDs (`CTX-01`, …) are public API: never renumber or reuse them.

## Do not touch

- `fixtures/` files unless you are deliberately changing what a maturity
  level means — they are test assertions, not examples to "improve".
- Version numbers are bumped by the release skill, not ad hoc.

---
> Source: [paladini/harness-score](https://github.com/paladini/harness-score) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
