## api-doc-v2c

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the OpenAPI 3.1.0 specification for the V2C Thirdparty API (EV charger remote control/management), plus the tooling to build interactive HTML documentation using Redocly CLI.

## Commands

**Preview (live dev server):**
```bash
npm run preview
# or: npx @redocly/cli preview-docs openapi.yaml
```

**Build (generate index.html):**
```bash
npx @redocly/cli build-docs openapi.yaml -t template.hbs -o index.html
```

> Note: The `npm run build` script in `package.json` omits the `-t template.hbs` flag. Always use the full command above to apply the custom template.

**Post-build step:** After building, the generated `index.html` references `redoc.standalone.js`. This must be replaced with the local `redoc.js` file, and any wrapper `<div>` injected by Redocly should be removed. Do this manually or via script after each build.

## Architecture

- **`openapi.yaml`** — The single source of truth. All API endpoints, schemas, parameters, security definitions, and inline `x-codeSamples` live here. This is the only file to edit when updating API documentation.
- **`template.hbs`** — Handlebars HTML template used by Redocly to wrap the generated Redoc output. Contains the page `<head>`, metadata, and footer. Uses `{{{redocHead}}}` and `{{{redocHTML}}}` placeholders.
- **`redoc.js`** — Custom local build of the Redoc standalone JS bundle. Replaces the CDN-hosted `redoc.standalone.js` in the output `index.html`.
- **`redocly.yaml`** — Redocly configuration: names the API `v2c_api`, enables cURL code samples only, and skips optional parameters in generated samples.
- **`index.html`** — The build artifact (generated file). Do not edit manually; it is overwritten on each build.

## API Structure

The `openapi.yaml` is organized into these tag groups:
- **Device Control & Configuration** — POST actions (start/stop charge, set parameters)
- **Device Status & Data Retrieval** — GET queries for device state
- **RFID Management** — RFID card registration/removal
- **Charging Statistics** — Session and aggregated charge history
- **Advanced Power Profiles** — Solar mode and custom power profiles
- **Firmware Management** — Firmware version queries
- **Pairing Management** — Device pairing for the user account

Authentication is via a custom `apikey` header (Bearer Token). The base server is `https://v2c.cloud/kong/v2c_service/`.

---
> Source: [V2Charge/API-doc-v2c](https://github.com/V2Charge/API-doc-v2c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
