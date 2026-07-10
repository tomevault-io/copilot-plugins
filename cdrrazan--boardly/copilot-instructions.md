## boardly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Boardly is a config-driven **GitHub Action** that automates GitHub Projects (v2): sprint rollover, stale-card nudges, sub-issue Done-gating with parent roll-up, sprint digests, daily standups, and priority auto-sort. It runs inside the adopter's own GitHub — no hosted infrastructure. A static marketing site lives in `web/`.

## Commands

```bash
npm run typecheck        # tsc --noEmit (strict, ES modules)
npm test                 # node:test via tsx, all test/*.test.ts
npm run build            # ncc bundle src/ -> dist/index.js (committed)
npm run all              # typecheck + test + build (run before committing)

# Single test file
node --import tsx --test test/features.test.ts
# Single test by name
node --import tsx --test --test-name-pattern="rollover" test/features.test.ts
```

**CI requires `dist/` to be in sync with `src/`.** After any change under `src/`, run `npm run build` and commit the regenerated `dist/`. CI fails if `git status --porcelain dist` is non-empty. `dist/` is a committed artifact, not gitignored — this is required for JS-based GitHub Actions.

## Architecture

Full detail in `docs/ARCHITECTURE.md` and `TECH_STACK.md`. The essentials:

- **Fetch once, act many.** `index.ts` reads Action inputs, loads/validates config, fetches the project a single time into an in-memory `ProjectGraph` (`types.ts`), then dispatches features over that snapshot. Keeps API usage low and makes features unit-testable without network.
- **Feature dispatch.** `index.ts` holds a `RUNNERS` map (feature key → `run<Feature>(ctx: RunContext)`) and `isEnabled`. The `only` input runs one feature; otherwise every enabled feature runs. **A failing feature is logged + marked failed but does not abort the others**; the audit trail is always flushed.
- **Config is a Zod schema.** `config.ts` is the single source of truth for config shape (with defaults); `loadConfig()` turns `.github/project-automation.yml` into a validated `Config` or a friendly error.
- **Fields referenced by name.** Users name their own Status/Priority/Iteration fields; config maps names, and `util/project.ts` resolves them to ids at runtime (`statusOf`, `iterationOf`, `isDone`, `optionId`, …). Missing fields produce an error listing what *is* available.
- **`dry-run` is centralized.** Features call `ctx.audit.record(...)` first, then check `ctx.dryRun` before mutating — so the audit trail is identical whether or not changes apply.
- **De-dup via hidden markers.** Nudges/gate comments embed an HTML comment marker; before commenting a feature scans existing comments and skips if it already acted during the current status "stint" (comment newer than the status's `updatedAt`).
- **"Time in status"** ≈ each field value's `updatedAt` from the Projects v2 API, not a timeline walk.

Data flows through the normalized `ProjectGraph`/`ProjectItem`/`ProjectField` model — downstream code never touches raw GraphQL JSON.

### Module map

| Module | Responsibility |
|--------|----------------|
| `src/index.ts` | Entry point: inputs → config → fetch → dispatch → flush audit. |
| `src/config.ts` | Zod schema + `loadConfig()` YAML loader. |
| `src/types.ts` | Normalized `ProjectGraph` model. |
| `src/github/` | `queries.ts` (GraphQL docs), `client.ts` (`ProjectClient`: pages project, normalizes, typed mutation/comment helpers, sends `sub_issues` header), `normalize.ts`. |
| `src/features/` | One file per feature. `context.ts` defines `RunContext`. Pure logic; side effects via `ctx.client`. |
| `src/notify/` | Slack (native `fetch`) + email (`nodemailer`) channels + `Notifier`. |
| `src/util/` | `audit.ts` (action accumulator + job-summary table, dry-run seam), `dates.ts`, `project.ts` (field accessors). |
| `test/helpers.ts` | Hand-built `ProjectGraph`s + `FakeClient` that records mutations. Feature tests run real `run*` against fabricated boards, assert on recorded calls. |

## Adding a feature

`features/<name>.ts` exporting `run<Feature>(ctx)` → extend the Zod schema in `config.ts` → register in `index.ts` `RUNNERS` + `isEnabled` → record every action through `ctx.audit`, honor `ctx.dryRun` → add tests + a `docs/use-cases/` page. Steps in `CONTRIBUTING.md#adding-a-new-feature`.

## The website (`web/`)

Plain HTML + CSS + vanilla JS, **no build step**, dark-only. Deployed to Cloudflare Pages via Git integration. Security headers in `web/_headers`.

---
> Source: [cdrrazan/Boardly](https://github.com/cdrrazan/Boardly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
