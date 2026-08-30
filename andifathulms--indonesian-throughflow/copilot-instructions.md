## indonesian-throughflow

> Indonesian Throughflow visualisation: particle advection over NASA OSCAR surface currents, with bathymetry as the mechanism and mooring-derived transports at the gates. Static site, GitHub Pages, no backend, no runtime network.

# CLAUDE.md — Arus Lintas

Indonesian Throughflow visualisation: particle advection over NASA OSCAR surface currents, with bathymetry as the mechanism and mooring-derived transports at the gates. Static site, GitHub Pages, no backend, no runtime network.

Read `PRD.md` before starting any task, and **`DESIGN.md` before writing any UI** — it is authoritative for visuals and it opens with the shared house layer used across these projects.

**Four things shape everything:**

1. **The chokepoints are below the grid.** OSCAR is 0.25° — roughly 27 km. Labani is 45 km wide, Lombok 35 km. The most important passages are one or two cells across, and published assessments show low-resolution products underestimate net flow there specifically. **Every gate narrower than two cells renders with a visible under-resolved flag**, and the numeric claims come from moorings, not from the grid.
2. **OSCAR is derived, and surface only.** Computed from altimetry, winds and SST through a simplified dynamical model, averaged over the top 30 m. Never present it as measurement, never as the full-depth throughflow.
3. **Particles never cross land.** A particle traversing an island is both a bug and a physical impossibility. Land is masked from advection; culls are matched by respawns.
4. **The advection is the product.** One orchestrated moment, per the house layer. Nothing else animates.

---

## Stack

- Next.js 14, App Router, `output: 'export'` — static only
- TypeScript, `strict: true`
- Tailwind CSS, tokens from `DESIGN.md`
- WebGL2 for particle state, canvas fallback
- Zod for manifest validation
- Vitest
- pnpm
- **No mapping library with a tile dependency, no charting library, no particle library.** The advection is the project.
- Fonts via `next/font`, self-hosted.

## Commands

```bash
pnpm dev
pnpm build                  # static export; runs data:validate first
pnpm preview                # serve ./out under the production basePath
pnpm test                   # vitest watch
pnpm test:run               # vitest once — before every commit
pnpm test:field             # decode integrity, sampling, quantisation round-trip
pnpm test:advect            # domain containment, land masking, population, determinism
pnpm test:gates             # citation completeness, under-resolved flags
pnpm data:fetch             # DEV/CI — pull OSCAR + bathymetry
pnpm data:build             # crop, quantise, PNG-pack, emit manifest
pnpm data:validate          # manifest, DOI, version, size budget
pnpm bench:particles        # particle budget + frame rate on a mid-range profile
pnpm typecheck
pnpm lint
```

`pnpm test:advect`, `pnpm test:gates` and `pnpm data:validate` gate the build and CI.

## Layout

```
app/
  [locale]/                 # id (default), en
    arus/                   # the field + controls + stops
    gerbang/[slug]/         # gate detail + section
    metode/                 # dataset, DOI, limitations, citations
components/
  field/                    # bathymetry + particle canvas
  dial/                     # month dial — cyclical
  enso/                     # three-state switch — categorical
  gate/                     # marker, flag, transport panel
  section/                  # cross-channel depth profile
  legend/                   # the honesty contract
  table/                    # text equivalent of the map
lib/
  field/                    # THE CORE. Pure. Runs in Node.
    decode.ts               # PNG-packed Uint8 → u,v via stated scale
    sample.ts               # bilinear interpolation
    mask.ts                 # land mask lookup
    streamline.ts           # integration, used for reduced-motion fallback
  advect/                   # particle state, seeding, culling, respawn
  gates/                    # passage definitions, mooring figures, citations
scripts/
  build-data.ts             # DEV/CI — netCDF + bathymetry → packed layers
data/
  field/                    # packed u/v layers, month × ENSO, + manifest
  bathymetry/               # packed depth grid + land mask
  gates/                    # five passages: width, sill, transport, citation
tests/
  field/  advect/  gates/
```

## Invariants

1. **`lib/field` is pure.** Typed arrays in, values out. No DOM, no React, no clock, no network, no module-level mutable state. Testable in Node.

2. **Land is masked from advection.** Every advection step checks the mask. A particle entering land is culled and respawned. **Never let a trail cross an island** — asserted by test.

3. **Particle population is conserved.** Every cull is matched by a respawn. Population drift is a bug.

4. **Advection is deterministic given a seed.** Seeded PRNG carried in state, never `Math.random`. Same seed, identical trajectory set.

5. **Particles never leave the domain.** Boundary handling is cull-and-respawn, never clamp — clamping produces particles piling up at the edge, which reads as a current that does not exist.

6. **Origin colouring is physical, not decorative.** North Pacific and South Pacific water carry distinct hues and converge across the Banda Sea. Do not assign origin colours arbitrarily or reuse them for anything else.

7. **Hue carries origin; speed is carried by trail length and opacity.** Never encode speed in hue.

8. **Every gate narrower than two grid cells renders an under-resolved flag.** Asserted by test. A gate rendering without its caveat is a correctness bug, not a cosmetic one.

9. **Every transport figure cites its mooring programme.** Validator-enforced. **No transport number is ever computed from the OSCAR grid** — the grid cannot resolve these passages and a number derived from it would be wrong in a way that looks authoritative.

10. **The legend always states: derived, surface-only, 0.25°, active month and ENSO state, and where the grid cannot resolve a passage.** Never collapsed by default. `DESIGN.md` §9.

11. **No real-time, forecast, or navigational framing** anywhere in data, code, copy or metadata. Climatology and composites only.

12. **No registration-gated datasets.** Copernicus GLORYS resolves the straits better and requires an account; excluded so the build stays open and reproducible. Do not add it.

13. **Raw netCDF is never committed.** The pipeline emits PNG-packed `Uint8` layers with a stated scale. Total shipped data under 2 MB.

14. **`--mooring` cyan is instruments only; the origin hues are particles only.** Warm means modelled field, cool-bright means measured. Do not mix. `DESIGN.md` §4.

15. **Reduced motion gets a complete static streamline field**, not a frozen frame and not an empty canvas.

16. **Zero network requests at runtime.** Attribution and DOI appear on the map and in the repository.

17. **Nothing is computed in a component.**

## Working style

- **Read `DESIGN.md` §1 first.** The house layer is shared across sibling projects; the palette, type and instruments are this app's own. Do not drift the shared parts.
- **Pipeline before render.** M0 settles layer shape and size.
- **Benchmark particles at M1, before any UI.** A stuttering field makes the advection unreadable, and the advection is the entire product. Test on a mid-range phone, not a laptop.
- **Ship the resolution statement at M2, with the legend** — not at M3 with the gates. A beautiful under-resolved map with no caveat is worse than no map.
- **When you want a number for a strait, go to the mooring literature.** Never compute it from the grid, however tempting.
- **When adding a control, ask what shape the data is.** Cyclical gets a dial, categorical gets a switch. `DESIGN.md` §7.
- **Don't touch `next.config.js`, the Actions workflow, `data:validate`, or the gate flag logic without saying so explicitly.**
- **Never weaken a test to make something pass**, especially `test:advect` or `test:gates`.

## Conventions

- Named exports; defaults only where Next requires them.
- Discriminated unions for layers, gates and results, keyed on `type`. Exhaustive `switch` with a `never` default.
- No `any`. No non-null `!` in `lib/field` or `lib/advect`.
- Velocities in m/s named `*Mps`. Transports in Sverdrups named `*Sv`. Depths and widths in metres named `*M`. Coordinates `[lon, lat]` WGS 84, named `*Lon` / `*Lat`.
- Gate ids stable and readable: `makassar`, `labani`, `lifamatola`, `lombok`, `ombai`, `timor`. They appear in URLs.
- Comments cite the dataset version or the mooring publication behind any figure.
- Indonesian first in UI copy; oceanographic terms in their standard form — *Arus Lintas Indonesia*, *Selat Makassar*, *Laut Banda*, *Sverdrup*.
- Tailwind tokens exactly as in `DESIGN.md` — `abyssal`, `deep`, `slope`, `shelf`, `land`, `contour`, `north-pacific`, `south-pacific`, `mooring`. Never raw hex in components.

## Testing rules

- `pnpm test:run` before every commit; `test:field`, `test:advect` and `test:gates` before any commit touching the pipeline, `lib/field` or `lib/advect`.
- Advection asserted: no particle leaves the domain; no particle enters masked land; population conserved; same seed produces identical trajectories.
- Bilinear sampling asserted against hand-computed values at cell corners and centres.
- Grid decode asserted against source metadata — dimensions, bounds, value range; quantisation round-trips within its scale.
- Gate flags asserted: every passage narrower than two cells renders its under-resolved marker.
- Citation completeness asserted on every transport figure.
- Legend asserted to state resolution, derivation, surface-only, and active state on every change.
- Reduced-motion streamline fallback asserted present and non-empty.
- `bench:particles` asserted against the budget and frame rate.
- Bug fix → failing test first.

## Deployment

`main` builds and deploys via Actions; data validation and the advection suite gate it. `basePath` must match the repository name; `.nojekyll` must exist in `out/`. Field and bathymetry ship as separate chunks. Verify with `pnpm preview` before pushing, and check the particle path on a real mid-range device before any release.

## Framing

The site states that it shows a multi-year climatology rather than current or forecast conditions, that the velocity field is derived from satellite altimetry, winds and SST rather than measured, that it represents the top 30 m only, and that the narrow straits are below the grid resolution with their numbers taken from moored observations. NASA OSCAR is cited with its DOI. Not a navigational product. No OIKN or government branding anywhere.

## Current state

M0 — not yet scaffolded. Next: OSCAR fetch, crop, quantise and PNG-pack, plus the bathymetry and land-mask layers and the manifest. **No render work until the layers validate and the size budget is met; no UI until `bench:particles` clears on a mid-range profile.**

---
> Source: [andifathulms/indonesian-throughflow](https://github.com/andifathulms/indonesian-throughflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
