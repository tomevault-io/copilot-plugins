## librarr

> Project memory for Claude sessions working in this repo.

# CLAUDE.md

Project memory for Claude sessions working in this repo.

## Project

**Librarr** — ebook/audiobook collection manager, the OpenLibrary-based
continuation of the archived Readarr (Servarr-family sibling of Sonarr,
Radarr, Lidarr). Forked from Readarr at upstream `develop` HEAD
`0b79d300` ("Retirement announcement", 2025-06-27). Currently at
**`1.0.0-beta`** — engineering gate cleared, see
[`CHANGELOG.md`](CHANGELOG.md). The previous upstream tagged release was
`v0.4.18.2805` (commit `7cc02f95`, 2025-06-10).

Full architecture map: **`ARCHITECTURE.md`** at repo root (including a
"Librarr fork additions" section), plus per-folder `README.md` files in
every major directory.

## Identity quirk (read this first)

The csproj/assembly names are `Readarr.*` but C# **namespaces are still
`NzbDrone.*`** — set deliberately in `src/Directory.Build.props:97-99` via
a `RootNamespace` rewrite. `using Readarr.Core;` will NOT compile —
`using NzbDrone.Core;` will. Don't "fix" this; it's intentional. The
`Stylecop.ruleset:1` file even still labels itself "Rules for Radarr"
from when the ruleset was forked over.

Also intentionally kept as `readarr`/`Readarr`:

- **On-disk identifiers** in
  `src/NzbDrone.Common/Extensions/PathExtensions.cs:15-26` — `readarr.db`,
  `readarr.restore`, `readarr_update`, `readarr_backup`,
  `readarr_appdata_backup`, `Readarr.Update`. Renaming would break
  `LegacyMigrationService` (expects `readarr.db` as input) and any
  existing install upgrading in place.
- **Binary names** produced by csproj — `Readarr.exe`,
  `Readarr.Console.exe`, macOS `CFBundleExecutable=Readarr`. These
  follow the csproj names above.
- **Cross-app icons** under
  `frontend/src/Content/Images/Icons/logo-{readarr,sonarr,radarr,lidarr,prowlarr}.png`
  — these display *other* Servarr family members in the UI, not us.

## Common commands

```bash
# Frontend
yarn install                                    # install deps (root package.json)
yarn start                                      # webpack --watch
yarn build                                      # one-shot → _output/UI
yarn lint                                       # ESLint
yarn stylelint-linux                            # Stylelint over CSS

# Backend
./build.sh --backend --enable-extra-platforms   # full backend build (multi-RID)
./test.sh                                       # backend tests
dotnet test src/NzbDrone.Core.Test/             # one test project
dotnet run --project src/NzbDrone.Console/      # run on :8787 (HTTPS :6868)

# Single test
dotnet test src/NzbDrone.Core.Test/ --filter "FullyQualifiedName~MyClassTests"
```

CI uses the same scripts. **Note:** StyleCop only enabled on the Linux CI
leg (`azure-pipelines.yml:79`); Mac/Windows skip it.

## Stack (develop HEAD versions)

- **Backend:** .NET 6 (`dotnetVersion: '6.0.427'`), ASP.NET Core, **DryIoc
  5.4.3** DI (`src/NzbDrone.Host/Bootstrap.cs:9-10,90`), custom Dapper-based
  ORM in `NzbDrone.Core/Datastore/`, Servarr-forked FluentMigrator
  (41 migrations), dual **SQLite + PostgreSQL**, NLog logging. Sentry 3.31
  for error reporting. Shipping version `1.0.0-beta`
  (`azure-pipelines.yml:22`). `Directory.Build.props:77`
  `AssemblyVersion 10.0.0.*` is the historical Readarr placeholder
  the CI overwrites at build time; not the shipping version.
- **Frontend:** React 17 + Redux 4 (**legacy `createStore`**, not RTK),
  Webpack 5, CSS Modules via PostCSS, `@microsoft/signalr`. Partial
  JS→TS migration (~985 `.js` / 375 `.ts` / 36 `.tsx` — ~29% TS).
  ~151 hook callsites alongside dominant class-component style.
- **Tests:** NUnit + Moq + FluentAssertions. Selenium 3.141 + ChromeDriver
  91 in `NzbDrone.Automation.Test` (years out of date — treat that suite
  as historical).

## Conventions

- **Strict build:** `TreatWarningsAsErrors=true`,
  `EnforceCodeStyleInBuild=true` (`src/Directory.Build.props:4-5`).
- **Backend file layout** per domain under `NzbDrone.Core/{Domain}/`:
  `Model/`, `{Entity}Repository.cs`, `{Entity}Service.cs`, `Commands/`,
  `Events/` (with `Handlers/`).
- **REST:** `Readarr.Api.V1/{Domain}/{Entity}Controller.cs` +
  `{Entity}Resource.cs` DTO. Manual mapping (no AutoMapper).
- **Frontend:** PascalCase folder per component, `Foo.js` + `Foo.css` +
  `FooConnector.js`. PropTypes for `.js`, TS types for `.ts/.tsx`
  (`react/prop-types: 2 / off` in `frontend/.eslintrc.js:317,365`).
- **Provider plugins** (indexers, download clients, notifications, import
  lists, metadata): all derive from a `ThingiProvider`-rooted base and
  are auto-discovered by DryIoc reflection — no manual registration.
- **Messaging:** `EventAggregator` (in-process pub/sub) + `CommandQueueManager`
  (DB-backed background queue). Handlers implement `IHandle<TEvent>` /
  `IExecute<TCommand>`.

## Where to add things

| Task | Location |
|---|---|
| New indexer | `src/NzbDrone.Core/Indexers/` — extend `HttpIndexerBase` |
| New download client | `src/NzbDrone.Core/Download/Clients/` — extend `TorrentClientBase` or `UsenetClientBase` |
| New notification | `src/NzbDrone.Core/Notifications/` — extend `NotificationBase` |
| New import list | `src/NzbDrone.Core/ImportLists/` — extend `HttpImportListBase` |
| New DB column | `src/NzbDrone.Core/Datastore/Migration/0XX_Name.cs` (FluentMigrator) + update model |
| New API endpoint | `src/Readarr.Api.V1/{Domain}/{Entity}Controller.cs` + `{Entity}Resource.cs` |
| New background job | Define `ICommand` + `IExecute<TCommand>` handler; schedule in `src/NzbDrone.Core/Jobs/TaskManager.cs` |
| New UI page | `frontend/src/{Feature}/` + route in `frontend/src/App/AppRoutes.js` (no lazy loading) |
| New Redux slice | `frontend/src/Store/Actions/{feature}Actions.js`, use the `Creators/` factories |

## Gotchas

- **Dual SQLite + Postgres:** every new query must be valid on both. Use
  `WhereBuilderSqlite` / `WhereBuilderPostgres` for predicate translation.
  Date/time types in particular have dedicated Postgres migrations.
- **`Parser/Parser.cs` is ~905 lines of regex** — most fragile file. A
  single careless change breaks many release-name patterns. There is no
  golden-corpus test fixture; treat with respect.
- **jQuery `$.ajax`** is the only HTTP client on the frontend
  (`frontend/src/Utilities/createAjaxRequest.js`). Don't introduce `fetch`
  or `axios` without a plan to migrate the whole layer.
- **`NzbDrone.Windows` vs `NzbDrone.Mono`** — platform shims, only one is
  active at runtime. Don't reference them directly from `Core/`; let DI
  pick via `OsInfo.IsWindows`.
- **`win-arm64` is intentionally NOT in the RID list**
  (`src/Directory.Build.props:11`). Windows-on-ARM is unsupported.
- **`config.xml` reload-on-change is disabled** (`Bootstrap.cs:237`) —
  changes to bootstrap config require a restart.
- **No pre-commit hooks** — lint runs in CI only.
- **Upstream Readarr is retired** — any changes here diverge from the
  rest of the Servarr ecosystem. Librarr is the continuation; the
  fork swaps `bookinfo.club` for native OpenLibrary as the primary
  metadata source. Do **not** add a `rreading-glasses` dependency —
  the fork is committed to direct OL.

## Process modes

`Bootstrap.GetApplicationMode` (`src/NzbDrone.Host/Bootstrap.cs:186-227`)
picks one of: `Help`, `RegisterUrl`, `InstallService`, `UninstallService`,
`Service` (Windows service), `Interactive` (tray/console). The
self-updater is a separate exe (`src/NzbDrone.Update/`).

## Default ports

- HTTP `8787`, HTTPS `6868` (`Bootstrap.cs:135-136`).
- Override via `config.xml` (`Port`, `SslPort`, `BindAddress`).

## Documentation here

- **`README.md`** — Librarr overview + "Migrating from Readarr" guide.
- **`CHANGELOG.md`** — Keep-a-Changelog format release notes.
- **`ARCHITECTURE.md`** — full code map; includes a "Librarr fork
  additions" section near the top that inventories everything new
  since upstream.
- **`MASTER-PLAN.md`** — strategic blueprint for the 12-phase revival
  (most of phases 0-11 are now shipped; 12 is post-1.0 backlog).
- **`METADATA-MIGRATION.md`** — historical sketch; superseded by
  the shipped `LegacyMigrationService` + `ReidentifyService`.
- **`src/*/README.md` and `frontend/src/*/README.md`** — per-folder
  signposts pointing back to `ARCHITECTURE.md` sections.

---
> Source: [Rorqualx/Librarr](https://github.com/Rorqualx/Librarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
