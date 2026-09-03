## historyviz

> Visualizes notable events in Earth's history on a linked map and timeline. See

# HistoryViz

Visualizes notable events in Earth's history on a linked map and timeline. See
`README.md` for the product intent; this file is the working map of the code.

## Commands

```bash
npm run dev            # dev server on :3000
npm test               # vitest, pure logic + corpus validation
npm run typecheck      # tsc --noEmit
npm run lint           # eslint
npm run build          # next build

npm run load:events -- --check    # parse + diff against Convex, no writes
npm run load:events               # sync changed markdown into Convex
npm run load:events -- --prune    # also delete records whose file is gone
npx tsx scripts/render-reconstruction.mts > /tmp/r.html   # visual plate check
```

## Two data backends

`src/lib/events/source.ts` picks one at request time:

- **Convex** when `NEXT_PUBLIC_CONVEX_URL` is set. The production path.
- **Markdown** otherwise — parses `data/events/**.md` on the server.

The fallback exists so a fresh clone runs with no deployment to configure. It
is not a second implementation: both go through the shared predicates in
`src/lib/events/filter.ts`, which `convex/events.ts` also imports (by relative
path — Convex bundles with esbuild and does not read the `@/` alias).

`convex/_generated/` does not exist until someone runs `npx convex dev`. Two
consequences: `convex/**` is excluded in `tsconfig.json`, and `source.ts`
reaches Convex through `anyApi` rather than the generated typed client.

## Where the substance is

| Concern | File |
|---|---|
| Rotation math (quaternions, slerp) | `src/lib/geo/rotation.ts` |
| Plate model + Euler stages | `src/lib/geo/plates.ts` |
| Dual coordinates (present ↔ paleo) | `src/lib/geo/reconstruct.ts` |
| Ring winding normalization | `src/lib/geo/winding.ts` |
| Basemap reconstruction | `src/lib/geo/land.ts` |
| Time scale, parsing, formatting | `src/lib/time.ts` |
| Event contract | `src/lib/events/schema.ts` |

## Things that will bite you

**Ring winding.** d3-geo reads a *clockwise* ring (in lon/lat order) as the
interior. A bounding box written the natural way — east, north, west, south —
is counterclockwise, so d3 draws the entire rest of the planet. Geometry is
normalized once in `parseEventMarkdown`; do not skip that path when adding a
data source. `coverageFraction` guards the case normalization cannot fix.

**Global events carry no geometry.** A polygon that circles the globe in
longitude is degenerate on a sphere. Such events declare `scope: global` and
have `geometry: null`; the map draws them as a frame, and their bbox is the
whole world so they match any viewport.

**The plate model is a placeholder.** ~10 plates, positions pinned at a handful
of ages and interpolated. It is right in outline — India crosses the Tethys,
the Atlantic closes by 200 Ma — and wrong in detail. Replacing it with a real
GPlates rotation file means editing `PLATES[].stages` and nothing else.

**Timeline coordinates must be rounded** before they become SVG attributes
(`px()` in `Timeline.tsx`). Full float precision serializes differently on
server and client, which React reports as a hydration mismatch.

## Conventions

- Years are astronomical integers: 1969, −2999, −66_043_000. `PRESENT_YEAR` is
  the reference for "ago".
- Coordinates are `[longitude, latitude]`, GeoJSON order, present-day frame.
- Event marks use one series colour, not one per category — eight categorical
  hues cannot be separated reliably as scattered map marks. Category is a
  filter. See the note at the top of `globals.css` before adding colours.

---
> Source: [dmeklund/historyviz](https://github.com/dmeklund/historyviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
