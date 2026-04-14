## xy-latlng-to-map

> This project is a Web Application designed to interactively map and convert coordinates. It primarily functions as a tool for:

# Project Context: xy-latlng-to-map

## Overview
This project is a Web Application designed to interactively map and convert coordinates. It primarily functions as a tool for:
*   Mutual conversion between XY coordinates and Latitude/Longitude.
*   Conversion between Japanese Geodetic System (Tokyo Datum) and World Geodetic System (WGS84).
*   Visualizing locations with markers and circles on a Leaflet-based map.
*   Measuring distances between points, markers, or circles.

The application leverages **Leaflet** for mapping, **Proj4** for coordinate projection, and **Vite** as the build tool. The codebase is written in **TypeScript**.

## Architecture & Technologies
*   **Frontend Framework:** Vanilla TypeScript with **Vite** for bundling.
*   **UI Library:** **Bootstrap 5** (SCSS).
*   **Mapping Engine:** **Leaflet** with various plugins:
    *   `leaflet-contextmenu`: Right-click menu actions.
    *   `leaflet-arc`: Drawing arcs.
    *   `leaflet-arrowheads`, `leaflet-geometryutil`.
*   **Data Handling:**
    *   `proj4`: Coordinate transformation.
    *   `jspreadsheet-ce`: Spreadsheet-like data input/display.
    *   `geo4326`: Geodetic calculations.
*   **Testing:** **Jest**.

## Key Directories & Files
*   `src/`
    *   `js/`: Core application logic (TypeScript).
        *   `map.ts`: Main map initialization and interaction logic.
        *   `leaflet.ts`: Leaflet configuration and tile layer setups (GSI tiles, etc.).
        *   `marker.ts`: Marker creation and management.
        *   `proj4.ts`: Projection definitions.
    *   `css/`: SCSS styling (Bootstrap customization).
    *   `partials/`: EJS templates for HTML components.
    *   `index.html`: Main entry point.
*   `test/`: Jest unit tests (e.g., `isvalidXY.test.js`).
*   `package.json`: Dependency management and scripts.
*   `vite.config.js`: Vite configuration.

## Development Workflow

### Setup
Ensure Node.js is installed.
```bash
npm install
```

### Running Development Server
Starts the Vite development server with hot module replacement.
```bash
npm run dev
```

### Building for Production
Compiles TypeScript and SCSS into static assets in `dist/`.
```bash
npm run build
```

### Running Tests
Executes Jest unit tests.
```bash
npm test
```

## Conventions
*   **Language:** TypeScript is used for logic. Ensure strict typing where possible (Note: existing codebase may have some loose typing or legacy JS patterns, as seen in mixed `.js` files in `test/` or `js_backup/`).
*   **Styling:** SCSS with BEM-like naming or Bootstrap utility classes.
*   **Map Data:** Uses GSI (Geospatial Information Authority of Japan) tiles by default.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/s-show)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/s-show)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
