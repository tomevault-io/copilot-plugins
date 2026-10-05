## kalup

> Kalup keeps a HubSpot portal's configuration (properties, groups, custom objects, later pipelines and association labels) in TypeScript files that the tool parses and never executes. Changes are reviewed as plans, applied to any named target portal, and edits made in the HubSpot UI are held as drift instead of reverted. The same files type the app with no generate step. This repo is the open-source core.

# Kalup: configuration as code for HubSpot

Kalup keeps a HubSpot portal's configuration (properties, groups, custom objects, later pipelines and association labels) in TypeScript files that the tool parses and never executes. Changes are reviewed as plans, applied to any named target portal, and edits made in the HubSpot UI are held as drift instead of reverted. The same files type the app with no generate step. This repo is the open-source core.

## Layout

- `packages/core`: `@kalup/core`, what user files and apps import. Codecs, `InferProperties`, `defineConfig`, `defineRemoved` and the config types. No other exports.
- `packages/engine`: `@kalup/engine`, private, bundled into the CLI. The config grammar reader and writer, the loader, the IR, plan and state contracts, classify, blueprints, the issues table and the JSON Schemas; the HTTP clients and endpoint registry, pull, the planner and the executor (`src/engine/`, `src/lib/`); the brand constant. No process, terminal, oclif or file system: hosts inject the state store, portal lock, journal and clock. `test/support/` holds the portal simulator and fixture helpers the CLI tests share.
- `packages/cli`: `kalup`, the CLI, bin `kalup`: the oclif commands, the terminal host, keys from the environment, and the file-backed state store, lock, journal and staged writes. It ships the user docs in `docs/` and the JSON Schemas as `kalup/schemas/<file>`.
- `packages/tsconfig`: shared TypeScript config, and `stamp.ts`, the build fingerprint both packages write into dist and the CLI tests check.
- `apps/web`: the website and docs site, `@kalup/web` (Fumadocs on Next.js).
- `examples/`: example projects, type-checked in CI.
- `scripts/conformance/`: the live conformance runner.
- `docs/`: [architecture.md](docs/architecture.md) (the design and its reasons), [compatibility.md](docs/compatibility.md) (what is stable), [hubspot.md](docs/hubspot.md) (HubSpot behaviour, the live journeys and the conformance runner).

Tooling: pnpm, turbo, biome (Ultracite), tsdown, vitest, changesets, oclif. Node 22.18+ to build, 22.13.1+ to run.

## Commands

- `pnpm build && pnpm check && pnpm test` passes today and must pass when you are done.
- `pnpm lint:fix` (Ultracite fix) before you finish; `pnpm lint` checks.
- `pnpm --filter <package> test` for one package. The CLI tests run the built CLI, so build first.
- `pnpm gen` after editing `packages/engine/src/issues.ts`. The error pages in `packages/cli/docs/errors/` and the website errors reference are generated from it, never edited by hand.

## House style

- No semicolons, single quotes, 120 columns, trailing commas. Function declarations. No default exports except config files.
- A `biome-ignore` needs a reason, and only for serial HubSpot requests, control-character handling, bit arithmetic where it is the point, and a tokenizer loop. `biome.jsonc` explains each override.
- Tests live under `packages/<pkg>/test/` mirroring `src/` (`src/codecs/builders.ts` is tested by `test/codecs/builders.test.ts`), fixtures with invented names under `test/fixtures/`.
- Check human output with inline snapshots; assert codes, exit codes and JSON data directly.
- Add a changeset only for a user-visible change to released behaviour.

## Vocabulary

Use the terms in `docs/architecture.md`. A portal in a project is a target, never an environment. A resource is addressed as `<type>:<path>`, for example `property:companies/billing_status`. Drift is held, not reverted. A tombstone written by `kalup rm` has `action: 'destroy'` or `'release'`.

## Hard rules

1. Never copy source from a client repository. Read it to understand behaviour, then write fresh.
2. Fixtures, examples, tests and docs use invented names. No client names, portal IDs or property names from client work.
3. The default test run never touches the network. Live tests are opt-in, gated by an environment variable, against an authorized developer test account.
4. Every HubSpot request goes through the endpoint registry; writes only through the write client.
5. Never print, log or commit a token, including in error and debug output. Never put a person's email address in a request header or payload.
6. Prose has no em dashes. Plain English, point first, no filler.
7. Do not publish to npm without the founder's explicit go-ahead. Commit and push only when asked.
8. Finish the assigned change, report validation and remaining work. When something is underspecified or looks wrong, stop and ask.
9. Check `docs/architecture.md` before re-arguing a settled decision.
10. Absence never deletes. The one exception is takeover mode (`docs/architecture.md` section 7), which also needs `allowDestroy` on the target. Nothing destructive runs without a person confirming it at a terminal.

## Escalate if

Stop and ask before a change that would:

- execute config, or load project code from the engine;
- let `--yes` or `--approve` cover a delete, or let any command prompt without a TTY;
- send a request outside the endpoint registry, or a write outside the write client's allowlist;
- advance the base where the approved and live values do not agree, or treat an incomplete read as proof of absence or equality;
- put a token, portal ID or transport name in the IR, or a token anywhere in output, state, plans or the journal;
- change the meaning of a JSON field, an exit code or an issue code.

Kalup is an independent open-source project maintained by Scopious. It is not affiliated with, endorsed by, or sponsored by HubSpot, Inc. HubSpot is a registered trademark of HubSpot, Inc.

---
> Source: [scopiousdigital/kalup](https://github.com/scopiousdigital/kalup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
