## ultra-agent-skill

> >


# Ultra Query Skill

Ultra is a web application for making maps with MapLibre GL JS using data from OpenStreetMap
and other sources. It extends the classic Overpass Turbo concept with modern rendering, YAML-based
configuration, and support for many query providers beyond Overpass.

The user's goal is typically: "I want to query some OSM data and see it styled nicely on a map."
Your job is to produce a complete Ultra query document they can paste into Ultra and run.

## Output Format

**Always output Ultra queries directly in the chat as code blocks.** Do not create files, save
to disk, or use any file-creation tools for Ultra queries — even for long or complex ones.
The user needs to copy-paste the query into Ultra, and a code block in the chat is the most
convenient format for that. Use a plain fenced code block (triple backticks with no language
tag) so the entire query document (YAML frontmatter + query body) is easy to select and copy.

## Anatomy of an Ultra Query Document

Every Ultra query is a single text document with two parts:

```
---
(YAML frontmatter — configuration and styling)
---
(Query body — Overpass QL, SQL, GeoJSON, etc.)
```

The YAML frontmatter is optional. A bare Overpass query works fine on its own. But the frontmatter
is where the power lives — it controls the map style, query provider, popup behavior, and more.

## Default Behaviors

Follow these unless the user says otherwise:

- **Query provider**: Overpass QL. Set `type: overpass` only if auto-detection might be ambiguous;
  otherwise omit `type` since Overpass is auto-detected.
- **Bounding box**: Ultra provides template variables for the current map viewport that work
  across all query providers:
  - `{{bbox}}` — the standard Overpass format (south,west,north,east)
  - Individual values: `{{s}}`, `{{n}}`, `{{e}}`, `{{w}}` (short) or
    `{{south}}`, `{{north}}`, `{{east}}`, `{{west}}` (long)
  - Composite strings: `{{wsen}}` is equivalent to `{{w}},{{s}},{{e}},{{n}}`
  For Overpass, use `[bbox:{{bbox}}];` as the first statement.
  For other providers (QLever, Postpass, GeoJSON APIs), use the individual or composite
  shortcuts to embed viewport coordinates in your query.
  If the user names a specific region, use an `area` filter (Overpass) or a relation-based
  spatial filter (QLever `ogc:sfContains`) instead.
- **Basemap**: Don't include `extends` in the style unless the user asks for a specific basemap
  or the visualization requires sandwiching layers into an existing style (via `beforeLayerId`).
  When sandwiching is needed, a good default is
  `extends: https://styles.trailsta.sh/openmaptiles-osm.json`.
- **Styling**: Start minimal. Get the data on the map with a clean, readable style. Don't add
  elaborate color ramps, glows, or complex expressions unless the user asks. You can always
  iterate.
- **Output mode**: Choose the right Overpass `out` statement for the geometry type needed:
  - `out center;` for nodes or when you only need point locations of ways/relations
  - `out geom;` when you need the full geometry of ways (lines, polygons)
  - `out body;` + `>;` + `out skel qt;` for recursive descent (rarely needed in Ultra since
    `out geom` is simpler)

## Interpreting OSM Tags

OSM tag values often carry meaning beyond their everyday English definitions. The tagging
scheme grew organically from British English conventions and community consensus, so a
thoughtful approach is needed when translating user requests into queries.

**`highway=*` is a great example.** In OSM, "highway" means any public right of way — not
just major roads. The tag spans motor vehicle roads, footpaths, cycleways, and more:

| User intent | Typical OSM tags |
|-------------|------------------|
| "Roads" (motor traffic) | `highway` ∈ {`motorway`, `trunk`, `primary`, `secondary`, `tertiary`, `unclassified`, `residential`, `service`, `living_street`} |
| "Paths / trails" (foot) | `highway` ∈ {`footway`, `path`, `steps`, `pedestrian`} |
| "Bike infrastructure" | `highway=cycleway` or roads with `cycleway=*` tags |
| "Sidewalks" | `highway=footway` + `footway=sidewalk`, or `sidewalk=*` on roads |
| "Tracks" (agricultural/forest) | `highway=track` (note: *not* railroad tracks!) |

When a user asks for "roads," don't query all `highway=*` — that would include footways.

**Many other tags are confusing, for example:**

- `highway=unclassified`: a minor through-road (British term), *not* "unknown type"
- `natural=water`: can be used for any water bodies, including man-made
- `name:etymology=*` is confusing because `name:*=*` tags are usually for language codes, e.g. `name:fr=*`

## YAML Frontmatter Reference

All keys are optional. Here are the ones you'll use most:

### `style`

Controls MapLibre styling. Can be a URL to a full style, or an object with these keys:

- **`extends`**: URL to a basemap style to build on top of.
- **`layers`**: Array of MapLibre layer definitions for styling your query results.

```yaml
style:
  extends: https://styles.trailsta.sh/openmaptiles-osm.json  # optional
  layers:
    - type: circle
      paint:
        circle-color: red
        circle-radius: 5
```

### `type`

Force a specific query provider. Default is `auto` (auto-detected). Common values:
`overpass`, `postpass`, `qlever`, `geojson`, `sparql`, `ohsome`.

See the "Query Providers" section below and the reference files for non-Overpass providers.

### `options`

MapLibre `MapOptions` passed to the constructor. Useful for setting initial view:

```yaml
options:
  zoom: 14
  center: [-77.47, 37.57]
  # or use bounds instead:
  bounds: [-77.65, 37.58, -76.93, 38.14]
```

### `server`

Override the default Overpass/SPARQL server:

```yaml
server: https://overpass.private.coffee/api/
```

### `popupTemplate`

Customize the click-popup using LiquidJS syntax:

```yaml
popupTemplate: "{{tags.name}} ({{type}}/{{id}})"
```

**Colon-containing keys**: Tag keys with colons (e.g. `building:material`, `addr:street`,
`roof:shape`) must use bracket notation — `{{tags["building:material"]}}`, not
`{{tags.building:material}}` — or LiquidJS will throw a `TokenizationError`.

Set to `false` to disable popups.

### `popupOnHover`

Show the popup on hover instead of click:

```yaml
popupOnHover: true
```

### `title` / `description`

Metadata shown in the UI:

```yaml
title: Bike Infrastructure
description: Cycle lanes, parking, and repair stations.
```

### `controls`

Add MapLibre controls:

```yaml
controls:
  - type: NavigationControl
    options:
      visualizePitch: true
    position: bottom-left
```

### `transform`

JavaScript module to mutate query results before rendering. Exports a default function
that receives and returns GeoJSON. Can import from CDNs like esm.sh or skypack.dev.

### `querySources`

Which sources respond to click-inspection. Default: `[ultra]`.

## Styling System

Ultra extends the MapLibre style spec to reduce boilerplate.

### Key Simplifications

1. **No `source` needed.** Layers without a `source` automatically target the query results.
2. **No `id` needed.** Layer IDs are auto-generated.
3. **Auto paint/layout sorting.** You can put paint and layout properties directly on the
   layer object — Ultra moves them into the correct sub-object. So instead of nesting
   `paint: { circle-color: red }`, you can write `circle-color: red` at the layer level.
4. **Sandwiching with `beforeLayerId`.** To insert a layer below an existing basemap layer,
   set `beforeLayerId` to the target layer's ID. This requires `extends` to be set.

### Layer Types

Use standard MapLibre layer types:

| Type | Use for | Key properties |
|------|---------|----------------|
| `circle` | Point data | `circle-color`, `circle-radius`, `circle-blur`, `circle-opacity` |
| `line` | Ways/lines | `line-color`, `line-width`, `line-dasharray`, `line-opacity` |
| `fill` | Polygons | `fill-color`, `fill-opacity`, `fill-outline-color` |
| `symbol` | Icons & text | `icon-image`, `icon-color`, `icon-size`, `text-field`, `text-size` |
| `heatmap` | Density viz | `heatmap-weight`, `heatmap-intensity`, `heatmap-color`, `heatmap-radius` |
| `fill-extrusion` | 3D polygons | `fill-extrusion-height`, `fill-extrusion-color` |

### Bundled Sprites (Icons)

Ultra bundles [Maki](https://github.com/mapbox/maki) and
[Temaki](https://github.com/rapideditor/temaki) icon sets as SDF sprites. Use them in
symbol layers:

```yaml
icon-image: maki:bicycle
icon-color: blue
icon-halo-color: white
icon-halo-width: 2
```

Format: `maki:<icon-name>` or `temaki:<icon-name>`. Since they're SDFs, you can recolor
them freely with `icon-color` and add halos.

**When choosing an icon, consult `references/icons.md`** — it lists all ~740 available icons
organized by theme (cycling, food, infrastructure, accessibility, etc.) so you can quickly
find a fit-to-purpose icon rather than falling back to generic shapes.

You can also set `icon-image` to an HTTPS URL to load a custom image icon.

### Data-Driven Styling with Expressions

Use MapLibre expressions for conditional styling based on feature properties (OSM tags).
In Ultra's YAML, expressions are written as YAML arrays:

```yaml
icon-image:
  - case
  - [ "==", [ get, amenity ], cafe ]
  - maki:cafe
  - [ "==", [ get, amenity ], restaurant ]
  - maki:restaurant
  - maki:circle
```

Common expression patterns:
- `[ get, <tag> ]` — get an OSM tag value
- `[ "==", <expr>, <value> ]` — equality test
- `[ "!=", <expr>, <value> ]` — inequality test
- `[ has, <tag> ]` — check if tag exists
- `[ "!", [ has, <tag> ] ]` — check if tag does NOT exist
- `[ geometry-type ]` — returns `Point`, `LineString`, or `Polygon`
- `[ case, <cond1>, <val1>, ..., <fallback> ]` — conditional
- `[ match, <input>, <label1>, <val1>, ..., <fallback> ]` — multi-match
- `[ any, <cond1>, <cond2>, ... ]` — logical OR for filters
- `[ all, <cond1>, <cond2>, ... ]` — logical AND for filters
- `[ coalesce, <expr1>, <expr2> ]` — first non-null value

**⚠️ CRITICAL: Quote operators in YAML filters!** YAML interprets `!` as a tag indicator,
causing parse errors. Always quote these operators when used in filter arrays:
- `"=="` not `==`
- `"!="` not `!=`
- `"!"` not `!`

```yaml
# ✗ WRONG - will cause YAML parse error
filter: [ all, [ !=, [ geometry-type ], Point ], [ !, [ has, name ] ] ]

# ✓ CORRECT - operators are quoted
filter: [ all, [ "!=", [ geometry-type ], Point ], [ "!", [ has, name ] ] ]
```

### YAML Anchors

For repeated style blocks, use YAML anchors (`&name`) and aliases (`*name`). You can also
merge with `<<: *name` and override specific keys:

```yaml
paint: &base_paint
  line-width: 3
  line-color: blue
# ...later...
paint:
  <<: *base_paint
  line-color: red  # override just this key
```

### Multiple Layers

A single query can have multiple style layers. This is how you style different geometry types
or tag values differently in one visualization. Order matters — later layers render on top.

A common pattern: one layer for lines/polygons, another for point icons:

```yaml
style:
  layers:
    - type: line
      filter: [ "==", [ geometry-type ], LineString ]
      paint:
        line-color: blue
        line-width: 3
    - type: symbol
      filter: [ "==", [ geometry-type ], Point ]
      icon-image: maki:circle
      icon-color: blue
```

### Filters

Layer-level `filter` uses MapLibre expression syntax to show only matching features:

```yaml
filter: [ "==", [ get, highway ], cycleway ]
filter: [ any, [ "==", [ get, shop ], bicycle ], [ "==", [ get, amenity ], cafe ] ]
filter: [ has, name ]
```

## Query Providers

### Overpass QL (default)

The standard query language for OpenStreetMap. See `references/overpass-cheatsheet.md` for
a compact reference of common patterns, tags, and output modes.

For full Overpass QL documentation: https://wiki.openstreetmap.org/wiki/Overpass_API/Overpass_QL

### Postpass

SQL-based querying of OSM data. Set `type: postpass` in the frontmatter. Postpass is in early
development and defaults to the GeoFabrik Postpass API v0.2.

**If the user asks for a Postpass query and you're unsure of the schema, read
`references/postpass.md` first**, then fetch the full schema doc linked there.

### QLever

SPARQL-based querying against the QLever osm-planet dataset. Set `type: qlever`.

**If the user asks for a QLever query, read `references/qlever.md` first**, then consult
the example queries linked there.

### PMTiles

Ultra has built-in support for PMTiles via the `pmtiles://` protocol. Set `type: vector`
(or `raster`) and use a `pmtiles://` URL as the query body.
Please see `references/pmtiles.md` for more information and examples.

### Other Providers

Ultra also supports: `geojson`, `kml`, `gpx`, `tcx`, `esri`, `raw`, `raster`, `vector`,
`dsv`, `sparql`, `sophox`, `ohsome`, `osmxml`, `osmjson`, `osmWebsite`, `osmWiki`,
`taginfo`, `javascript`.

For details on any of these, consult: https://overpass-ultra.us/docs/yaml/

### Multiple Sources

You can also add additional
sources (PMTiles, raster tiles, etc.) via `style.sources`, allowing multiple tilesets
in one map. Layers target these extra sources with an explicit `source` key.
**If the user asks about adding multiple tile sources, or combining tilesets, read
`references/multiple-sources.md`** for syntax and examples.

## Worked Examples

### Simple: Coffee shops as circles

```
---
style:
  layers:
    - type: circle
      circle-color: '#6F4E37'
      circle-radius: 6
      circle-stroke-color: white
      circle-stroke-width: 1
---
[bbox:{{bbox}}];
nwr[amenity=cafe];
out center;
```

### Icons with Maki sprites

```
---
style:
  layers:
    - type: symbol
      icon-image: maki:bus
      icon-color: '#4A90D9'
      icon-halo-color: white
      icon-halo-width: 3
---
[bbox:{{bbox}}];
node[highway=bus_stop];
out;
```

### Mixed geometry: lines + points

```
---
style:
  layers:
    - type: line
      filter: [ "!=", [ geometry-type ], Point ]
      paint:
        line-color: '#E88D2A'
        line-width: 3
    - type: symbol
      filter: [ "==", [ geometry-type ], Point ]
      icon-image: maki:circle
      icon-color: '#E88D2A'
---
[bbox:{{bbox}}];
(
  way[footway=sidewalk];
  node[kerb];
);
out geom;
```

### Area filter: within a named region

```
---
style:
  layers:
    - type: symbol
      icon-image: maki:drinking-water
      icon-color: '#2196F3'
---
area["name"="Washington, D.C."]->.dc;
node[amenity=drinking_water](area.dc);
out;
```

## Tips

- **Bbox template variables**: Ultra replaces `{{bbox}}`, `{{s}}`, `{{n}}`, `{{e}}`, `{{w}}`
  (and long forms `{{south}}`, `{{north}}`, `{{east}}`, `{{west}}`) with the current viewport
  coordinates before sending any query. The short forms can be combined: `{{wsen}}` produces
  `<west><south><east><north>`. Use these for non-Overpass providers that need viewport coords.

- **Quoting tag keys with special characters**: Keys containing colons or other special chars
  need quotes in Overpass QL: `way["cycleway:right"=lane]`. In `popupTemplate`, use bracket
  notation: `{{tags["addr:street"]}}` (dot notation causes a `TokenizationError`).
- **`out center` vs `out geom`**: Use `out center` when you want point representations
  (smaller response, works well with symbol/circle layers). Use `out geom` when you need
  the actual line/polygon shapes.
- **Debugging**: If no data appears, the query might be too restrictive or the bbox too small.
  Suggest the user zoom out or check tag values on the OSM wiki.
- **Performance**: Large bbox + common tags (like `building=yes`) can return huge datasets.
  Warn the user if a query might be heavy.
- **`nwr` shorthand**: `nwr` means "node, way, or relation" — a convenient shorthand when
  you want all element types matching a filter.
- **Semicolons**: Every Overpass statement ends with `;`. A missing semicolon is the most
  common syntax error.
- **Union blocks**: Use `( ... );` to combine multiple queries. The results are merged.

## Ultra Examples Catalog

These examples on the Ultra docs site demonstrate creative combinations of queries and MapLibre
styling. **When the user asks for an advanced visual effect or technique you're not sure how
to implement, fetch the relevant example page** — each one contains a complete, working query
document you can learn from.

| Example | URL |
|---------|-----|
| Add a title to a map | https://overpass-ultra.us/docs/Examples/add-a-title/ |
| Add a custom modal control | https://overpass-ultra.us/docs/Examples/add-an-info-modal/ |
| Load GeoJSON with custom BBOX format | https://overpass-ultra.us/docs/Examples/alt-bbox-format/ |
| ATP USPS Dropboxes | https://overpass-ultra.us/docs/Examples/atp-usps-dropboxes/ |
| Bike Infrastructure | https://overpass-ultra.us/docs/Examples/bike-infra/ |
| User Contribution Heatmap | https://overpass-ultra.us/docs/Examples/contribution-heatmap/ |
| Create a custom map style | https://overpass-ultra.us/docs/Examples/custom-style/ |
| Emojis (flag sprites) | https://overpass-ultra.us/docs/Examples/emoji/ |
| **Glowing effect for points** | https://overpass-ultra.us/docs/Examples/glow-effect/ |
| HTMLControl link button | https://overpass-ultra.us/docs/Examples/htmlcontrol-linkbutton/ |
| Local landmarks map | https://overpass-ultra.us/docs/Examples/landmarks-prototype/ |
| map=yes | https://overpass-ultra.us/docs/Examples/map%3Dyes/ |
| Minimalist Ski Map | https://overpass-ultra.us/docs/Examples/minimalist-ski-map/ |
| 3D style prototype | https://overpass-ultra.us/docs/Examples/opentrailstash-3d/ |
| Overture Landcover + Hillshade | https://overpass-ultra.us/docs/Examples/overture-landcover-plus-hillshade/ |
| Combine Overture + OSM POI | https://overpass-ultra.us/docs/Examples/overture-places-and-osm/ |
| **Postpass** | https://overpass-ultra.us/docs/Examples/postpass/ |
| **QLever** | https://overpass-ultra.us/docs/Examples/qlever/ |
| Replace a basemap layer | https://overpass-ultra.us/docs/Examples/replace-a-layer-with-overpass-data/ |
| Sophox | https://overpass-ultra.us/docs/Examples/sophox/ |
| Style by opening_date | https://overpass-ultra.us/docs/Examples/style-by-openingdate/ |
| **Tree Cover** (stylized circles) | https://overpass-ultra.us/docs/Examples/trees/ |
| Viewing features at low zooms | https://overpass-ultra.us/docs/Examples/viewing-features-at-low-zooms/ |
| Wikidata photo popup | https://overpass-ultra.us/docs/Examples/wikidata-photo/ |
| Wikidata SPARQL | https://overpass-ultra.us/docs/Examples/wikidata-sparql/ |
| Features within admin boundary | https://overpass-ultra.us/docs/Examples/within-bounds/ |

Bolded entries are especially useful as reference for common requests.

## When You're Unsure

If the user asks for something and you're not confident about the exact syntax or capability:

1. For **Overpass QL** questions, read `references/overpass-cheatsheet.md` or consult
   https://wiki.openstreetmap.org/wiki/Overpass_API/Overpass_QL
2. For **MapLibre styling** questions (expressions, layer properties, paint vs layout),
   consult the official MapLibre Style Spec: https://maplibre.org/maplibre-style-spec/
3. For **Ultra-specific styling** (auto paint/layout, sprites, sandwiching, extends),
   consult https://overpass-ultra.us/docs/style/
4. For **YAML config** questions, consult https://overpass-ultra.us/docs/yaml/
5. For **Postpass/QLever**, read the relevant file in `references/` first.
6. For **advanced visual techniques** (glows, heatmaps, 3D, clustering, custom icons),
   fetch the relevant example from the catalog above — they contain complete working queries.
7. For **Ultra examples** generally, browse https://overpass-ultra.us/docs/Examples/

---
> Source: [MapRVA/ultra-agent-skill](https://github.com/MapRVA/ultra-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
