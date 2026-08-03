## geobim-busan

> The local machine may contain a stale GeoBIM PostgreSQL in Ubuntu WSL. It can

# Agent Instructions

## Absolute Database Identity and Safety Rule

The local machine may contain a stale GeoBIM PostgreSQL in Ubuntu WSL. It can
accept valid credentials while returning an old, internally consistent
dataset. A successful TCP connection or `/health` response is therefore not
proof that the correct database is in use.

- Before database, backend, Docker, migration, seed, restore, project-list, or
  borehole work, run `scripts/db-safety-check.ps1`.
- The only approved local API is `http://127.0.0.1:9001`. Do not use
  `localhost:9001`; an IPv6 `::1` listener can shadow the Docker IPv4 port.
- The only approved application DB is the `postgres` service in the
  `geobim-stratum` Docker Compose project.
- Never use host/WSL `localhost:5432` for this application.
- Never start host uvicorn as a fallback for the Docker backend.
- Before any database mutation, run `scripts/backup-db-safe.ps1`.
- Never run `docker compose down -v`, remove DB volumes, drop/recreate the DB,
  truncate core tables, or seed merely because a UI/API list is empty.
- First diagnose API URL, authentication, query filters, pagination, timeout,
  `/health/db`, and direct Docker counts.
- Production UI code must never catch an API/DB failure and silently return
  mock, fixture, seed, cached example, or hard-coded project/borehole data.
  Show an explicit connection error containing the active API URL instead.
- User workspaces 9707, 9708, and 9718 are required DB identity sentinels.
  Public source survey names must remain `public_source_legacy` and must never
  be returned by the default project-list API. Missing sentinels
  mean "wrong/stale DB"; they do not authorize restore or seed.
- Do not weaken the sentinel health check to make startup pass.

Canonical startup is `scripts/start-backend-safe.ps1`. See
`docs/DB_SAFETY_RUNBOOK.md`.

## Absolute Stratigraphic Modeling Rule

When changing the 3D stratum modeling code, treat borehole contact elevations as hard design constraints.

For every valid borehole and every observed stratum contact:

- The generated stratum boundary surface must pass through the borehole contact elevation at the borehole XY location.
- "Close enough because the interpolated grid is smooth" is not acceptable for design review.
- Grid-node interpolation error, bilinear sampling error, smoothing, mesh subdivision, pinch-out carving, voxelization, or visual post-processing must not move an observed contact away from its borehole elevation.
- If a rendering mode cannot satisfy this constraint, it must be clearly marked as approximate and must not be used as the design-authoritative surface.
- Any refactor must keep an automated diagnostic or test that reports the maximum borehole contact error. The target for the authoritative smooth surface is zero within floating-point tolerance.

Context: design firms regard a stratum surface that does not exactly pass through the measured borehole contact as a stratigraphic design error.

Relevant implementation area:

- `sites/viewer-3d/src/workers/geoWorker.ts`
- `sites/viewer-3d/src/lib/geoGeometry.ts`
- `sites/viewer-3d/src/hooks/useGeoModel.ts`

See also:

- `docs/stratum_contact_hard_constraint.md`

## Absolute Groundwater Observation Rule

Groundwater observations are design-authoritative hard constraints, independent
from the stratigraphic stack.

For every groundwater observation included in an authoritative groundwater
surface:

- The groundwater surface at the borehole XY must equal the observation head
  elevation exactly within floating-point tolerance.
- The authoritative elevation is
  `borehole_elevation - measured_groundwater_depth_bgl`.
- Smoothing, grid sampling, bilinear interpolation, terrain clipping,
  extrapolation, confidence fading, vertical exaggeration, mesh generation, or
  visual effects must never move the surface away from the observation at that
  XY.
- Groundwater must not be added to `strata_group`, layer-order clamping,
  pinch-out, basement extension, or watertight stratum-solid generation.
- If conflicting observations at the same XY are selected for one surface, the
  model must reject the input or require an explicit time/record selection. It
  must not silently average design-authoritative values.
- Every implementation or refactor must retain an automated off-grid test and
  a maximum observation-error diagnostic.
- A rendered or exported surface that fails the constraint must be marked
  approximate and must not be treated as design-authoritative.

See also:

- `docs/groundwater_observation_hard_constraint.md`

---
> Source: [jsunny-ai/geobim-busan](https://github.com/jsunny-ai/geobim-busan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
