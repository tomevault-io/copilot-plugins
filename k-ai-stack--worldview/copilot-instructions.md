## worldview

> python -m http.server 8080   # Python 3

# WorldView — Claude Code Instructions

## Running locally

```bash
python -m http.server 8080   # Python 3
# OR
npx serve .                  # Node.js
```

Then open http://localhost:8080. No build step — plain HTML/CSS/JS.

**Required before first run:**
Copy `src/config.example.js` → `src/config.js` and add a Cesium Ion token (free at ion.cesium.com). Nothing will render without it.

## Architecture

Single-page app. No framework, no bundler. Everything is vanilla JS loaded via `<script>` tags in `index.html`.

```
index.html          — layout, layer toggle rows, preset buttons, script tags
styles/main.css     — all styling
src/
  config.js         — API keys (gitignored, never commit)
  config.example.js — template (committed)
  main.js           — Cesium viewer init + click handler + layer event router
  ui/
    controls.js     — display mode switching (NVG/FLIR/CRT), layer toggle state
    presets.js      — camera flyTo bookmarks (LOCATIONS map)
  layers/
    *.js            — each layer is a self-contained IIFE: { enable(viewer), disable(viewer) }
```

## Layer pattern

Every layer file exports `WV.layers.layername = { enable(viewer), disable(viewer) }`.

`enable()` must return a Promise. `disable()` is synchronous.

The layer router in `main.js` listens for `wv:layerToggle` events and calls `enable`/`disable`. Status bar and count badge are updated via `WV.Controls.setStatus(msg)` and `WV.Controls.updateCount(layerName, n)`.

## Click-to-inspect

`main.js` handles `LEFT_CLICK` via `Cesium.ScreenSpaceEventHandler`. Picked objects expose their data via:
- `id._wvType` — string identifier (e.g. `'flight'`, `'port'`, `'seismic'`)
- `id._wvMeta` — `[{key, val}, ...]` array rendered in the intel panel
- `id._wvLat` / `id._wvLon` — used for zoom-on-click behaviors

For PointPrimitive and Billboard layers, `id` is a plain object attached at add-time. For Entity layers (polylines, billboards added via `viewer.entities`), `_wvType` and `_wvMeta` are set directly on the entity object.

## Key Cesium notes

- `requestRenderMode: true` — scene only redraws when explicitly requested. Always call `viewer.scene.requestRender()` after mutations.
- Viewer created in `main.js` and exposed as `window.WV.viewer` for layers that need it.
- 2D mode: `viewer.scene.morphTo2D(1.0)` — after 1.15s setTimeout, call `camera.setView({ destination: Rectangle.fromDegrees(-180,-85,180,85) })` to prevent vertical squish.
- Star background: procedural canvas injected behind Cesium WebGL canvas (requires `contextOptions: { webgl: { alpha: true } }`). Hidden in 2D mode.

## Config keys

| Key | Layer | Source |
|---|---|---|
| `CESIUM_TOKEN` | Globe rendering | ion.cesium.com (required) |
| `AISSTREAM_KEY` | Maritime vessels | aisstream.io (free) |
| `OPENSKY_USER/PASS` | Commercial flights | opensky-network.org (optional) |
| `SENTINEL_INSTANCE_ID` | Satellite imagery | dataspace.copernicus.eu (free) |

## Common tasks

**Add a new layer:**
1. Create `src/layers/newlayer.js` following the IIFE pattern with `enable`/`disable`
2. Add `<script src="src/layers/newlayer.js"></script>` to `index.html` (before `main.js`)
3. Add a `.layer-row` toggle row in `index.html` with `data-layer="newlayer"`
4. Add a `<span class="layer-count" id="count-newlayer">0</span>` badge

**Add a camera preset:**
Add an entry to `LOCATIONS` in `src/ui/presets.js`, then add a `<button class="preset-btn" data-preset="key">LABEL</button>` in `index.html`.

**Change default view / mode:**
- Default camera: `WV.Presets.flyTo(viewer, 'key')` in `src/main.js`
- Default display mode: `setMode('normal')` in `src/ui/controls.js`

---
> Source: [K-AI-STACK/WorldView](https://github.com/K-AI-STACK/WorldView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
