## first-js

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a JavaScript learning project focused on geological fault trace and seismicity visualization using OpenLayers. It's a simple client-side web application that displays USGS fault data and earthquake information on a 2D map without build tools or complex bundling.

## Code Preservation Rules

**NEVER remove commented-out code.** The codebase contains intentionally commented sections (like AK fault functionality) that must be preserved for future use. Only comment out additional code when explicitly requested - never delete commented blocks.

## Architecture

### Core Structure

- **Static Web App**: Traditional HTML/CSS/JS served via HTTP server
- **OpenLayers 8.2.0**: 2D mapping library via CDN (no local dependencies)
- **GeoJSON Data**: Multiple fault datasets and earthquake data loaded from external APIs
- **Single Page**: All functionality contained in index.html + main.js

### Key Files

- `index.html`: Entry point with OpenLayers CDN imports and UI controls
- `js/main.js`: OpenLayers map initialization, data loading, and interactions
- `css/style.css`: Map styling and UI panel layouts

## Features

### Map Layers

- **Base Maps**: USGS National Map hillshade with OpenStreetMap overlay
- **Fault Datasets**:
  - 2023 USGS NSHM (red, 50% opacity, 5px width)
  - UCERF3.1 (blue, 50% opacity)
  - UCERF3.2 (green, 50% opacity, dashed)
  - AK fault data (orange, commented out)
- **Seismicity**: USGS earthquake data with loading overlay and smart timeout

### Interactive Features

- **Layer Controls**: Toggle fault layers and earthquake data with centralized color config
- **Address Search**: Nominatim-powered location search with blue star markers
- **Feature Info**: Click on faults/earthquakes to view properties in sidebar
- **Custom Earthquake Queries**: Date range picker with 60s timeout warning and user choice to continue/cancel
- **Map Refresh**: Button to refresh rendering and reload current layer state

## Development

### Running the Application

```bash
# Serve files via HTTP server (required due to CORS)
python -m http.server 8000
# or
npx serve .
# or
php -S localhost:8000
```

Then navigate to `http://localhost:8000`

### Data Sources

- **USGS Fault Data**: Multiple endpoints via corsproxy.io
- **AK Fault Data**: fault-viewer-v3.arkottke.org API
- **Earthquake Data**: USGS earthquake feeds and FDSNWS Event Web Service
- **Geocoding**: Nominatim OpenStreetMap search API

### Technical Details

- **Projections**: Data loaded in EPSG:4326, displayed in EPSG:3857
- **CORS Handling**: Uses corsproxy.io for cross-origin requests
- **Styling**: Centralized color configuration in `LAYER_COLORS` object
- **Hit Detection**: 10px tolerance for feature clicking
- **Loading Management**: AbortController with smart timeout handling for slow API requests

### Current Limitations

- No build system or package management
- No testing framework or linting tools
- Relies on external CORS proxy for some data sources
- USGS API can be slow/overloaded for large date ranges

## Future Development Considerations

The project is set up for potential expansion with a comprehensive .gitignore that anticipates Node.js tooling, but currently operates as a minimal learning environment for JavaScript and OpenLayers fundamentals.

---
> Source: [asarmy/first-js](https://github.com/asarmy/first-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
