## umage-optimizely-graphsearchtools

> You are reading the **GraphSearchtools** addon for Optimizely CMS 12 / 13.

# AGENTS.md

You are reading the **GraphSearchtools** addon for Optimizely CMS 12 / 13.
This file orients AI agents that are helping a developer wire the addon
into a customer project. Read it before opening any other file.

## If you are integrating this addon

Start at **[`docs/integrator/quickstart.md`](docs/integrator/quickstart.md)** —
install, register a search channel, and see a telemetry event in the
Insights tool in about ten minutes. Then pick one of the three integration
patterns:

- [Server-rendered ASP.NET](docs/integrator/pattern-server-rendered.md) —
  Razor / MVC host that emits telemetry in-process.
- [Headless deployment](docs/integrator/pattern-headless.md) — SPA / SSR
  frontend that beacons telemetry to the addon's public ingest endpoint.
- [3rd-party telemetry](docs/integrator/pattern-third-party-telemetry.md) —
  decorate `ITelemetrySink` to fan out, or replace `ITelemetryReader` to
  source aggregates from an external warehouse.

The same four docs are packed inside the NuGet package at `/docs/` so
they are available even when reading the `.nupkg` offline.

## Tools that make integration sanity-checkable

- **JSON schema** for the `UmageAI:GraphSearchTools` config block lives at
  [`src/GraphSearchtools/schemas/graphsearchtools.schema.json`](src/GraphSearchtools/schemas/graphsearchtools.schema.json)
  and is packed in the `.nupkg` at `schemas/`. Reference it from the
  host's `appsettings.json` with a `"$schema"` line — see the sample site
  for an example.
- **Live config sanity check** at `GET /EPiServer/cms/graphsearchtools/Overview/Health`
  (auth: any role in `AuthorizedRoles`). Returns structured JSON with
  registered channels, credential status, feature toggles, telemetry
  state, and a `diagnostics[]` array describing problems an agent should
  fix. Treat this endpoint as the source of truth when verifying a setup
  — do not rely on inferring success from the absence of a 404.
- **Startup logs.** The addon writes a single info line at boot
  (`GraphSearchtools ready: N channel(s) registered, M diagnostic(s).`)
  followed by one warning or info per diagnostic. Tail these when
  smoke-testing a fresh install.

## Source-of-truth conventions

The public API surface (`AddGraphSearchtools`, `IGraphSearchtoolsBuilder`,
`SearchChannelBuilder`, `GraphSearchtoolsOptions`, `ITelemetrySink`,
`ITelemetryReader`, `ITelemetryMetrics`, `LocalTelemetryOptions`,
`FeatureToggles`, …) carries XML doc-comments that are the authoritative
reference. The integrator docs link into those `.cs` files rather than
restating them. **Do not mirror XML comment content into markdown when
you are extending the docs** — link to the file.

## Other files in this repo

- [`CLAUDE.md`](CLAUDE.md) — project structure, build commands, multi-target
  rules. Read when contributing to the addon itself, not when integrating it.
- [`docs/personas.md`](docs/personas.md), [`docs/design-system.md`](docs/design-system.md) —
  product design context. Out of scope for integrators; relevant if you are
  extending the addon's UI.
- [`docs/research/`](docs/research/) — background research on Optimizely
  Graph capabilities, authentication, and relevancy. Out of scope for
  integrators.

---
> Source: [umage-ai/Umage.Optimizely.GraphSearchtools](https://github.com/umage-ai/Umage.Optimizely.GraphSearchtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
