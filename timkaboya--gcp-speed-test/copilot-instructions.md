## gcp-speed-test

> Backendless **Angular 22** app that measures browser→Google Cloud region latency. Most work is

# Copilot instructions — gcp-speed-test

Backendless **Angular 22** app that measures browser→Google Cloud region latency. Most work is
in `ui/`. `AGENTS.md` is the authoritative conventions doc — this file is the fast, code-grounded
orientation for Copilot. Read `AGENTS.md`, `README.md`, and `CONTRIBUTING.md` for the full picture.

## The one architectural fact that shapes everything

GCP has **no per-region public storage endpoint** (Cloud Storage is a global anycast behind the
Google Front End). Per-region latency can only be measured by pinging a **region-pinned URL** of a
tiny HTTP responder deployed per region. Two modes exist:

- **Default (free):** reuses [gcping](https://github.com/GoogleCloudPlatform/gcping)'s public Cloud
  Run endpoints, baked into `ui/src/assets/data/endpoints.json`. They send **no CORS**, so the app
  pings with `fetch(url, { mode: 'no-cors' })` → **opaque responses**: timeable, but status code and
  resource timing are unreadable.
- **Production upgrade:** deploy our own `responder/` (Go) which returns permissive CORS +
  `Timing-Allow-Origin`, unlocking real status codes and resource timings. See README "Phase 0".

## Repo layout

| Path | What it is |
|------|-----------|
| `ui/` | Angular app — nearly all work happens here |
| `responder/` | Dependency-free Go 1.22 HTTP responder (`main.go`), one per region |
| `infra/` | Terraform: Artifact Registry + per-region Cloud Run (`main.tf`, `variables.tf`) |
| `.github/workflows/` | `ci.yml` (lint → test:ci → build), `deploy-responders.yml`, `azure-static-web-apps.yml` |

### `ui/src/app` code map

- `services/` — `region.service.ts` (baked-in region data + geography grouping), `seo.service.ts`,
  `theme.service.ts`. Barrel-exported via `services/index.ts`.
- `pages/gcp/latency/` — the core UI: `latency.component.ts` runs the ping loop and holds the
  single `LatencyState` (a `Map` of `RegionPingData` storing raw ping history; metrics are computed).
  `cloudflare-meta.store.ts`, `connection-details.component.ts` support it.
- `pages/information/`, `pages/privacy/`, `pages/shared/` — content + shared page pieces.
- `shared/` — reusable components (`copy-button`, `export-csv-button`, `footer`, `nav-groups`,
  `icons/lucide-icons`, `theme/theme-toggle`, `support-developer`), `constants.ts`, `utils.ts`.
- `models/` — `region.ts`, `theme.ts`, `assistant.ts` (barrel `models/index.ts`).
- Routing is lazy + capitalized path segments: `Gcp/Latency` (default), `Information`, `Privacy`
  (`app.routes.ts`). SSR entry points: `main.server.ts`, `server.ts`, `app.config.server.ts`.

## Commands (run from `ui/`)

```bash
npm install
npm start          # dev server → http://localhost:4200
npm run build      # production SSR/prerender build → static output
npm test           # unit tests (Vitest via @angular/build:unit-test builder)
npm run test:ci    # tests + coverage with enforced thresholds (what CI runs)
npm run lint       # angular-eslint
npm run format     # prettier --write .
node scripts/generate-endpoints.mjs   # regenerate src/assets/data/endpoints.json
```

- **Run a single test:** the runner is Vitest, but the Angular `unit-test` builder has no simple
  file-filter flag — focus with `describe.only(...)` / `it.only(...)` in the target `*.spec.ts`, then
  `npm test`. Remove the `.only` before committing.
- **Responder** (`responder/`): `go build ./...`, `go test ./...` (Go 1.22, stdlib only).
- **Infra** (repo root): `terraform -chdir=infra init|apply|destroy` with `-var project_id=...`
  `-var image=...`.

## Conventions that aren't obvious from one file

- **Angular 22, standalone + signals only.** `signal`/`computed`, `@Injectable({ providedIn: 'root' })`,
  `loadComponent`/`loadChildren` lazy routes, `ChangeDetectionStrategy.OnPush`. **No NgModules.**
- **Prettier (enforced):** no semicolons, single quotes, 2-space indent, `printWidth` 100,
  `trailingComma: none`, always-parens arrows. Imports auto-sorted by `@ianvs/prettier-plugin-sort-imports`
  — don't hand-order imports.
- **SSR/prerender aware:** region data is baked in and loaded **synchronously** in `RegionService`
  (async fetch caused a large CLS layout shift). Any browser-only work — above all the actual ping —
  must be guarded (`isPlatformBrowser(inject(PLATFORM_ID))`), as `latency.component.ts` does. Keep
  this in mind for anything touching startup or `RegionService`.
- **Tests are mandatory.** Every module has a `*.spec.ts`. `test:ci` enforces coverage thresholds in
  `ui/angular.json` (`test > configurations > ci`): statements/branches/functions **60**, lines **65**.
  Add/adjust tests for every change; never drop below thresholds.
- **Responder:** stay dependency-free (stdlib only). Every response sets CORS + `Cache-Control: no-store`
  + `Timing-Allow-Origin` via `setCommonHeaders`.

## Workflow / review (branch-protected)

- **`main` is protected — no direct pushes.** Every change is a PR from a feature branch.
- Requires **code-owner review** (`@timkaboya`, `.github/CODEOWNERS`) **and** green CI
  (`ci.yml`: lint → `test:ci` → build).
- Commits **signed off** (`git commit -s`); merges are **squash merge**. See `CONTRIBUTING.md` +
  `SECURITY.md`.

## Gotchas

- `endpoints.json` is generated at **build time** and cannot be fetched at runtime (gcping = no CORS).
- `no-cors` responses are opaque — real status/timing needs our own responders.
- **Legacy Azure names** linger despite this being a GCP app: `serve:ssr:azure-speed-test` in
  `package.json`, the `azure-speed-test` project key + `buildTarget`s in `angular.json`, and the
  `@azure/storage-blob` dependency. Don't be misled — the app is GCP.

---
> Source: [timkaboya/gcp-speed-test](https://github.com/timkaboya/gcp-speed-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
