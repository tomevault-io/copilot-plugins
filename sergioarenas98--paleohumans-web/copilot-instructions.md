## paleohumans-web

> - PaleoHumans is a public scientific web platform for European Upper Palaeolithic sites with human remains, built as the digital companion to Arenas del Amo et al. (2024), DOI `10.1016/j.jasrep.2024.104391`.

# AGENTS

## Project
- PaleoHumans is a public scientific web platform for European Upper Palaeolithic sites with human remains, built as the digital companion to Arenas del Amo et al. (2024), DOI `10.1016/j.jasrep.2024.104391`.
- Repo scope is an Angular CLI multi-project workspace with two SSR apps:
  - `projects/web`: public research site
  - `projects/backoffice`: admin/backoffice
- Current redesign/planning docs focus on `projects/web`. Backoffice is usually out of scope unless the task explicitly says otherwise.

## Architecture
- Angular 21 standalone components; no NgModules.
- Backend is an external Spring Boot REST API at `http://127.0.0.1:8080/api`.
- Public web is read-only and uses public GET endpoints.
- Backoffice uses JWT Bearer auth for writes and admin-only endpoints.
- Models/enums are duplicated in `web` and `backoffice`; there is no shared library.

## Domain Essentials
- Primary browsing hierarchy: `Site -> OsteologicalUnit -> Individual / Specimen -> Bone / Skeleton / Dates`.
- Supporting entities: `Culture`, `Reference`, `BurialGroup`, `BoneCatalog`, `DatingTechnique`.
- `Culture.color` and `Culture.colorRgb` are authoritative visual identifiers in the docs.
- `graveGoods`, `boneProcessing`, and `tracesOfOchre` are boolean flags, not descriptive text.
- Public dating data comes through `Specimen.dates[]`.
- Docs state there is no calibrated range field in the API; only uncalibrated BP (`datesBpUncal`) and range/sigma (`datesRange`) are documented.

## Implementation Guardrails
- Preserve SSR. Guard browser-only APIs (`window`, `navigator`, `IntersectionObserver`, Leaflet setup) with `isPlatformBrowser`.
- Treat `projects/web` and `projects/backoffice` as separate UI systems. Redesign docs propose `--dt-*` tokens for web and leaving backoffice `--t-*` tokens untouched.
- Do not assume aggregate count fields exist on `Site`; docs repeatedly note counts may need client-side aggregation or a dedicated backend endpoint.
- When touching route data fetching, consult the resolver audit first; SSR/hydration behavior is a documented sensitive area.
- Docs disagree on some current routes/resolvers/pages. Verify current code before changing navigation or data-loading flows.

## Commands
- `ng serve --project=web`
- `ng serve --project=backoffice`
- `ng build --project=web`
- `ng build --project=backoffice`
- `ng test --project=web`
- `ng test --project=backoffice`
- `ng lint --project=web`

## Docs Priority
- Product/domain overview: `docs/paleohumans-discovery/project-overview.md`
- Public-web domain mapping: `docs/paleohumans-discovery/domain-model-for-public-web.md`
- Backend/API truth: `docs/paleohumans-discovery/backend-api-map.md`
- Corrections to older discovery notes: `docs/paleohumans-discovery/backend-validation-notes.md`
- SSR/resolver guidance: `docs/architecture/resolver-audit.md`
- Public-web redesign spec: `docs/paleohumans-redesign-v3/chosen-direction.md`, `docs/paleohumans-redesign-v3/design-system.md`, `docs/paleohumans-redesign-v3/page-reinvention.md`, `docs/paleohumans-redesign-v3/rebuild-plan.md`

## Docs With Drift
- `docs/CODEMAPS/*` are generated summaries; use them for orientation, not final truth.
- `docs/context/Database Scheme.txt` is useful for raw schema context but conflicts with newer backend-validated docs in places.
- Older discovery/design notes can be outdated; prefer the backend validation docs and resolver audit when they conflict.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SergioArenas98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
