## twisted

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`twisted` is a TypeScript wrapper for the Riot Games API (League of Legends, Teamfight Tactics, Riot Account, and Data Dragon). It is published to npm as a library — there is no application to run; the deliverable is the compiled `dist/`.

## Commands

The package manager is **yarn** (`yarn.lock` is committed). Node 16 works for development; the published `engines` field claims `>=8.6.0`.

```bash
yarn install              # install deps (no node_modules is checked in)
yarn build                # tsc -> dist/ (also runs automatically on prepublishOnly)
yarn lint                 # eslint ./src/**/*
yarn lint:fix             # eslint --fix
yarn jest                 # run the full test suite (alias: yarn test)
yarn jest test/base.test.ts           # run a single test file
yarn jest -t "keep query params"      # run tests matching a name
RIOT_API_KEY={key} yarn example [fnName]   # run examples in runExamples.ts (ts-node)
```

- Coverage is **always collected** (`collectCoverage: true` in `jest.config.js`); there is no separate flag needed, but `yarn test:coverage` also exists.
- `test/live.test.ts` is excluded from normal runs (`testPathIgnorePatterns: ['/test/live']`). It hits the real Riot API and needs a valid `RIOT_API_KEY` + a real account in `.env`. Do not expect it to run in CI/offline.
- `testRegex` matches `*.test.ts`/`*.spec.ts` in **both** `test/` and `src/` (e.g. `src/constants/regions.test.ts` is a real test).

## Architecture

### Entry points → service composition

Three top-level classes are the public API, exported from `src/apis` and re-exported by `src/index.ts` (alongside `Constants` and `Dto`):

- `RiotApi` — account endpoints (region-group based)
- `LolApi` — League of Legends
- `TftApi` — Teamfight Tactics

Each entry class **composes service classes as `public readonly` fields** (see `src/apis/lol/lol.ts`), passing `this.getParam()` so every service shares the same config (key, retry settings, concurrency, debug). Example: `api.Summoner.getByPUUID(...)`, `api.MatchV5.list(...)`. Deprecated services are kept and marked `@deprecated` (e.g. `Match` vs `MatchV5`, `Spectator` vs `SpectatorV5`).

### Class hierarchy

```
BaseApi<Region>           src/base/base.ts          ← all HTTP logic lives here
  └─ BaseApiLol           game = 'lol',  Region = Regions
  └─ BaseApiTft           game = 'tft',  Region = RegionGroups | Regions
  └─ BaseApiRiot          game = 'riot', Region = RegionGroups | Regions
        └─ each service (SummonerApi, MatchV5Api, ...) extends one of these
```

The game-specific base only overrides the `game` field, which becomes the `:game` segment of the URL. The `<Region>` generic is what constrains, at the type level, whether a service takes platform `Regions` or `RegionGroups`.

### The request pipeline (`src/base/base.ts`)

`BaseApi.request<T>(region, endpoint, params?, forceError?, queryParams?)` is the single choke point for every Riot call. Understand the two distinct parameter bags:

- **`params`** (`IParams`) = **URL path** values. They are substituted into the endpoint's `path` placeholders written as `$(name)`, and `region` is injected automatically.
- **`queryParams`** = the **query string**, passed straight to axios as `options.params`.

URLs are built in `getApiUrl()` from declarative endpoint definitions in `src/endpoints/endpoints.ts` (`{ path, prefix, version }`), producing:
`https://$(region).api.riotgames.com/:game/{prefix}/v{version}/{path}`.

**Rate limiting & retries:** on `429`/`503`, `retryRateLimit()` waits (honoring the `retry-after` header) and re-issues the request up to `rateLimitRetryAttempts` times. When editing this path, the retry must forward **both** `params` and `queryParams` to the re-issued `request()` — dropping `queryParams` here is exactly the class of bug that issue #167 fixed.

**Concurrency:** `RequestBase` (`src/base/request.base.ts`) wraps axios in a `promise-queue`. Concurrency is set per `setConcurrency()` (default `Infinity`); the queue is a static singleton on `RequestBase`.

### Adding/extending an endpoint

The repeating pattern across the codebase:
1. Declare the endpoint in `src/endpoints/endpoints.ts` (under `endpointsV4`/`endpointsV5`/etc.), with `$(placeholder)` tokens in `path`.
2. Add a method to the relevant service in `src/apis/<game>/<service>/`, calling `this.request<TheDto>(region, endpoint, params, false, query)`. Path values go in `params`; query string goes in the 5th arg.
3. Add the response DTO under `src/models-dto/...` and export it from the nearest index so it surfaces under `Dto`.
4. If it's a new service, wire it into the game entry class (`lol.ts` / `tft.ts` / `riot.ts`).

### Data Dragon is the exception

`DataDragonService` (`src/apis/lol/dataDragon/DataDragonService.ts`) does **not** extend `BaseApi`. It calls axios directly against the static ddragon CDN — **no API key, no rate-limit queue, no retry logic**. Don't assume it shares the request pipeline.

## Conventions & gotchas

- **Regions vs RegionGroups** (`src/constants`): platform regions (e.g. `KOREA`, `NA1`) vs routing groups (`AMERICAS`, `ASIA`, `EUROPE`). Match-V5 and Account use region *groups*; most LoL endpoints use platform *regions*. The base class generic enforces which one a service accepts.
- **API key resolution**: the constructor reads `process.env.RIOT_API_KEY` unless a key is passed explicitly (string or `{ key }`). `.env` is loaded via `dotenv`.
- **`UPDATE_CHAMPION_IDS=true`** env var enables periodic refresh of champion IDs at runtime (for long-running processes).
- **Unit tests mock `internalRequest`**, not the network: `api.internalRequest = jest.fn().mockImplementationOnce(...)` lets you simulate responses/errors and assert on the axios `options` (including retry behavior). Follow this pattern instead of real HTTP.
- **Publishing**: `.npmignore` ships only `dist/`, `package.json`, and `yarn.lock`. `tsconfig.json` excludes `test`, `example`, and `runExamples.ts` from the build.
- eslint config (`.eslintrc.cjs`) turns **off** `@typescript-eslint/no-explicit-any`; `any` is used deliberately in the request layer.

---
> Source: [justadev-afk/twisted](https://github.com/justadev-afk/twisted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
