## dataset-helper-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dataset Helper Plugin for Climweb (Django/Wagtail framework). The plugin provides wizards to quickly create WMS reference datasets for the Climweb mapviewer, bootstrapping climate data platforms with pre-configured map layers.

**Domain Model:**
- **Category** → **SubCategory** → **Dataset** → **WmsLayer** → **WmsRequestLayer**
- Datasets are WMS-based and belong to categories/subcategories
- The plugin bulk-creates these objects from WMS endpoint capabilities
- The models are defined in `/__temp/models.py` and `./__tmp/wms.py`

## Architecture

**Dual-stack application:**
- **Backend**: Django plugin at `plugins/dataset_helper_plugin/` - Wagtail admin integration, API endpoints
- **Frontend**: Vue 3 SPA at `plugins/frontend/` - WMS layer discovery and selection UI

**Communication**: Vue frontend uses `window.postMessage()` to send selected layers to Django backend, which creates Dataset/WmsLayer objects via AJAX to `/admin/dataset_helper/action/`.

**Key Models** (from geomanager library):
- `Dataset`, `Category`, `SubCategory`, `WmsLayer`, `WmsRequestLayer`, `Metadata`


**Important**
You must not consider the frontend.
When asking for code, just generate backend code in the plugin folder.

## Plugin Vision

The plugin acts as a **managed catalog** of WMS layers for Climweb:

1. **Ships a default layer catalog** (~119 layers from various sources: JRC, ECMWF, CAMS, CGLS, etc.) defined in a JSON config
2. **Admin selects layers** via a collapsible tree UI (Category > SubCategory > Layer) with checkboxes — all checked by default
3. **"Load Layers" provisions** checked layers into Climweb DB (Category, SubCategory, Dataset, WmsLayer, Metadata)
4. **Tracks sync state** between the catalog and Climweb via the `CatalogEntry` model:
   - Each catalog entry has a `product_code` (natural key) and a nullable `dataset_id` (loose UUID ref to Climweb Dataset)
   - Status: `synced` (green), `pending_add` (orange), `pending_remove` (red), `disabled` (gray)
5. **Supports custom layers**: admin can add layers manually or import from a remote WMS GetCapabilities
6. **Origin tracking**: each entry is tagged as `config`, `manual`, or `wms_import`

## Development Commands

### Backend (from `plugins/dataset_helper_plugin/`)

### Docker
```bash
docker compose build    # Build dev image (requires climweb_dev:latest base)
docker compose up -d    # Run all services
# Access at http://localhost/admin
```

## Key Files

| Purpose             | Path                                                                                      |
|---------------------|-------------------------------------------------------------------------------------------|
| API Views           | `plugins/dataset_helper_plugin/src/dataset_helper_plugin/views.py`                        |
| Wagtail Integration | `plugins/dataset_helper_plugin/src/dataset_helper_plugin/wagtail_hooks.py`                |
| Admin Template      | `plugins/dataset_helper_plugin/src/dataset_helper_plugin/templates/dataset_helper_plugin/index.html` |
| mapviewer models    | `__tmp/models.py` |
| mapviewer wms model | `__tmp/wms.py` |

## WMS Defaults

When creating datasets, the plugin uses:
- Version: 1.3.0
- Format: image/png
- SRS: EPSG:3857
- Tile size: 256x256
- Multi-temporal: enabled

---
> Source: [fgg-consultant/dataset-helper-plugin](https://github.com/fgg-consultant/dataset-helper-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
