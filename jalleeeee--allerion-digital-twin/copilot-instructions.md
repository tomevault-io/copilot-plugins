## allerion-digital-twin

> This is **Allerion Intelligence's Critical Mineral Digital Twin** — a browser-based 3D geological visualization platform built for federal procurement (USGS/DOD). It consists of two pages:

# CLAUDE.md — Allerion Digital Twin Developer Context

## Project Overview
This is **Allerion Intelligence's Critical Mineral Digital Twin** — a browser-based 3D geological visualization platform built for federal procurement (USGS/DOD). It consists of two pages:

- **index.html** — Project Genesis landing page (corporate portal, timeline, document verification)
- **twin.html** — Interactive 3D Digital Twin powered by CesiumJS

## Architecture
- index.html - Landing page (self-contained HTML + CSS + JS)
- twin.html - 3D Digital Twin shell (loads Cesium + app.js)
- css/styles.css - Twin-specific styles
- js/app.js - Core Cesium application logic (~750 lines)

## Tech Stack
- CesiumJS 1.114 — loaded via CDN
- Pure HTML/CSS/JS — no frameworks, no build step
- Cesium Ion — terrain, imagery, and KML assets
- Hosting — GitHub Pages

## Key Coordinates
Property Center: 37.453466, -88.374611 (Section 19, T12S, R8E, Hardin County, IL)

## Critical Rules
- All mine shafts MUST be within the 40-acre property boundary
- Hicks Dome data is NOT on-property — always label as ADJACENT
- REE claims below 310ft are inferred, not measured on-site
- Color palette: Black #040506, Olive Drab #5a6850, Silver #9da2a6

## Adding Cesium Sandcastle Features
The Sandcastle viewer maps to our viewer variable in app.js. Copy JS code into app.js, push entities to the layers array, and add toggle buttons in twin.html sidebar.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jalleeeee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
