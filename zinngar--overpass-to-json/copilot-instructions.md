## overpass-to-json

> This project is a single-page application built with Vite and TypeScript. It allows users to fetch traffic light data from the Overpass API and download it as a JSON file.

# Traffic Light Data Fetcher

This project is a single-page application built with Vite and TypeScript. It allows users to fetch traffic light data from the Overpass API and download it as a JSON file.

## How it works

1. The user clicks the "Fetch Traffic Lights Data" button.
2. The application sends a POST request to the Overpass API interpreter with a query for all nodes tagged with `highway=traffic_signals`.
3. The fetched data is stored in memory.
4. The user clicks the "Download JSON" button to download the data as a `traffic_lights.json` file.

## Verification

To verify the project:
1. Run `npm run build` to ensure there are no TypeScript or build errors.
2. Check the `dist` folder for the generated production build.

---
> Source: [zinngar/Overpass-to-Json-](https://github.com/zinngar/Overpass-to-Json-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
