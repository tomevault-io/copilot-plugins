## qgis-plugin

> QGIS plugin that connects to the [Infrared City](https://infrared.city) simulation platform. Distributed via the QGIS plugin repository and GitHub Releases. The plugin code is open source (GPL-2.0+); access to the simulation backend requires a subscription.

# Infrared City GIS — QGIS Plugin

QGIS plugin that connects to the [Infrared City](https://infrared.city) simulation platform. Distributed via the QGIS plugin repository and GitHub Releases. The plugin code is open source (GPL-2.0+); access to the simulation backend requires a subscription.

## Stack

- Python 3 (whatever QGIS ships — typically 3.9+)
- QGIS 3.44 – 3.x (PyQGIS / PyQt5) — QGIS 4 (Qt6) not yet supported; `metadata.txt` caps at `qgisMaximumVersion=3.99`
- `pb_tool` for plugin packaging (`infrared_city_gis/pb_tool.cfg`)
- Internal services: `infrared-sdk`, REST calls to `api.infrared.city`

## Repository Layout

```
qgis-plugin/
├── infrared_city_gis/         # The QGIS plugin (this is what gets shipped)
│   ├── __init__.py            # Plugin entry point — classFactory()
│   ├── infrared_city_gis.py   # Main plugin class
│   ├── infrared_city_*.{py,ui}# Dialogs (auth, fetch geometry, ground materials, simulation, bbox, trees)
│   ├── client.py              # HTTP client wrapper around infrared-sdk
│   ├── services/              # Domain helpers (fetch, area_poller, geometry, buildings)
│   ├── models/                # Analysis, vegetation, time-frame parsers
│   ├── visualization/         # Raster rendering helpers
│   ├── utils/                 # Shared utilities
│   ├── icons/                 # Toolbar icons (PNG/SVG)
│   ├── i18n/                  # Translations
│   ├── metadata.txt           # QGIS plugin metadata (version, deps, tags)
│   ├── pb_tool.cfg            # Build config for pb_tool
│   └── requirements.txt       # Python deps (installed at runtime)
└── .github/workflows/         # CI: release builds the ZIP on tag push
```

The plugin **must** ship as a single folder (`infrared_city_gis/`) zipped at the root — that's what QGIS expects when users install from ZIP.

## Common Commands

```bash
# Build a plugin ZIP locally (mirrors what CI does on tag push)
# Excludes caches, hidden files, and the dev-only tests/ dir so the package
# stays clean for plugins.qgis.org (no hidden-file warnings).
zip -r infrared-city-qgis.zip infrared_city_gis/ \
  -x "*__pycache__*" "*.pyc" "*.pyo" "*.DS_Store" "*/.*" \
     "infrared_city_gis/tests/*" "infrared_city_gis/test/*"

# Lint
pylint --rcfile=infrared_city_gis/pylintrc infrared_city_gis/
```

Plugin uploads to `plugins.qgis.org` are **manual via the web UI** — see [`docs/deployment.md`](docs/deployment.md).

**Note:** `infrared_city_gis/pb_tool.cfg` exists but is currently stale — it references files that have been moved or renamed. Don't run `pb_tool zip` until the config is updated to match the current layout.

## Release Process

Triggered by pushing a `v*` tag (see `.github/workflows/release.yml`):

```bash
# Bump version in infrared_city_gis/metadata.txt first, commit, then:
git tag v0.2.2 && git push --tags
```

CI builds the ZIP and creates a GitHub Release. Upload to `plugins.qgis.org` is still **manual** — review the release on plugins.qgis.org before promoting to non-experimental.

See [`docs/deployment.md`](docs/deployment.md) for full deploy details.

## Doc Map

- [`README.md`](README.md) — user-facing overview, install, and usage.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — contributing guide (canonical filename; there is no `CONTRIBUTION.md`).
- [`docs/architecture.md`](docs/architecture.md) — component overview, dialog flow, API contract.
- [`docs/vegetation-input.md`](docs/vegetation-input.md) — tree-layer input contract (OSM-native: `species`/`genus`/`leaf_type`, optional size; two-tier resolution — precise registry species or archetype; catalog override). Only the point geometry is mandatory.
- [`docs/ground-materials.md`](docs/ground-materials.md) — ground-material (surface) layers: fetch dialog, `ground-*` layer convention, simulation usage.
- [`docs/manual-testing.md`](docs/manual-testing.md) — manual/exploratory test checklist for every dialog function (API key, fetch, ground materials, simulation matrix, trees).
- [`docs/battle-scars.md`](docs/battle-scars.md) — non-obvious gotchas and workarounds (PyQGIS, pb_tool, plugin distribution).
- [`docs/deployment.md`](docs/deployment.md) — how to cut a release, plugins.qgis.org review process.
- [`docs/release-process.md`](docs/release-process.md) — Release Please flow (staging → main → tag).

## License & Distribution

- **GPL-2.0-or-later** — required because plugins link against PyQGIS (also GPL).
- Distributed via plugins.qgis.org (preferred — gets discoverability) and GitHub Releases (fallback).
- The repo is `qgis-plugin`, the plugin display name is **Infrared City GIS**, and the inner package folder is `infrared_city_gis/` (must stay underscore-named — Python module requirement).

---
> Source: [Infrared-city/qgis-plugin](https://github.com/Infrared-city/qgis-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
