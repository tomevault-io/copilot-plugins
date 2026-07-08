## controledeopcoes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ControleDeOpcoes** is a Google Apps Script (GAS) web application for managing a Brazilian stock options portfolio (B3 exchange). It consists of:
- A **GAS backend** (`.gs` files) that reads/writes Google Sheets and calls external APIs
- A **Vue 3 frontend** (`.html` files) assembled at runtime by GAS's `include()` template system

There is no build step, no `package.json`, and no local dev server. All execution happens inside Google's Apps Script runtime.

## Deployment Workflow

### Automatic (CI/CD) — preferred

**Every push to `main` automatically deploys to the GAS DEV project** via GitHub Actions (`.github/workflows/deploy-gas-dev.yml`). The job runs `clasp push --force`, then updates the web app deployment (reusing the existing one — GAS caps scripts at 20 versioned deployments) and reads the **real web app URLs from the Apps Script API** (`entryPoints[].webApp.url`); they appear in the step summary and are committed to `.webapp-urls` (key `HEAD_URL` = always-latest code). Never hand-construct `/macros/s/<id>/dev` URLs. Manual trigger: **Actions → Deploy to GAS DEV → Run workflow**.

Full setup guide (reusable for other projects): `docs/GUIA_CICD_GITHUB_GAS.md`.

Key files:
- `.clasp.json` — points to the DEV scriptId (committed; scriptId is not a secret)
- `.claspignore` — excludes non-GAS files (`pwa-mobile/`, `mockups/`, `docs/`, `.github/`, `*.md`). **Any new non-GAS file/folder must be added here**, otherwise GAS tries to execute it and crashes (e.g. a PWA `sw.js` caused `ReferenceError: self is not defined`)
- GitHub Secret `CLASPRC_JSON` — clasp OAuth credentials (classic `"token"` + `"oauth2ClientSettings"` format, **not** the clasp v3 `"tokens"` format)

### Manual (fallback)

```bash
# Push local changes to GAS
clasp push

# Open the GAS editor
clasp open

# Deploy as web app (done from the GAS IDE: Deploy > Manage Deployments)
```

Running individual backend functions (tests, syncs) is done by selecting the function name in the GAS editor and clicking **Run**.

## Running Tests

Each `.gs` module has a dedicated integration test function. Run them from the GAS editor by selecting the function and clicking Run:

| File | Test Function |
|---|---|
| `000_CoreServiceAPIClient.gs` | `testSuiteApiClient()` |
| `001_CoreServiceConfig.gs` | `testConfigArchitectureV5()` |
| `002_CoreDataUtils.gs` | `testSuiteDataUtilsV2()` |
| `004_CoreServiceLogger.gs` | `testSuiteLoggerV3()` |
| `005_CoreServiceUI.gs` | `testSuiteUIHandler()` |
| `006_CoreOrchestrator.gs` | `testSuiteOrchestrator()` |

To validate the full infrastructure stack at once, run `testeFinalIntegridade()` in `Código.gs`.

## Required Script Properties

Set these in **GAS Project Settings > Script Properties**:

- `OPLAB_ACCESS_TOKEN` — OPLab API authentication token
- Claude API key — configured via `025_ConsultorIAClaudeSonnet45.gs`

## Architecture

### Backend Layer (`.gs` files — numbered by load order)

Files are numbered `000`–`025`; GAS loads them in alphabetical order, so numbering enforces dependency order.

**Infrastructure (000–005) — must be loaded before any engine:**

| File | Singleton | Responsibility |
|---|---|---|
| `000_CoreServiceAPIClient.gs` | `ApiClient`, `OplabService` | HTTP fetch with retry/backoff; OPLab API adapter |
| `001_CoreServiceConfig.gs` | `SYS_CONFIG`, `ConfigManager` | Sheet name map, Universal Data Dictionary (DUD), 3-layer config cache |
| `002_CoreDataUtils.gs` | `DataUtils` | Header maps, row maps, merge helpers, date/float parsing |
| `003_CoreSanitizador.gs` | `Sanitizador` | Input sanitization (BRL currency, pt-BR dates, tickers) |
| `004_CoreServiceLogger.gs` | `SysLogger`, `DataExtractorService` | Buffer-then-flush logging to LOGS sheet; cockpit/asset extractors |
| `005_CoreServiceUI.gs` | `UIHandler` | Silent bridge pattern for all menu-triggered functions |

**Orchestration:**

- `006_CoreOrchestrator.gs` — `CoreOrchestrator`: reads `Orquestrador_Sequencia_Padrao` and `Orquestrador_Sequencia_OPLab` keys from `Config_Global` sheet (semicolon-separated function names) and executes them sequentially
- `Código.gs` — Entry points: `onOpen()` (builds menu), `doGet()` (serves the web app), `include()` (template slot system)

**Sync Engines (007–022):**

- `007_CoreUpdatePortfolio.gs` — syncs NECTON_IMPORT → COCKPIT
- `008–009` — stock price data and 250-day history
- `010–011` — options data and options history
- `012–013` — Greeks (via OPLab API and native Black-Scholes)
- `014–022` — OPLab market data (series, best rates, volumes, variations, rankings, correlations, fundamentals)

**AI Module (025):**

- `025_ConsultorIAClaudeSonnet45.gs` — Claude Sonnet 4.5 portfolio advisor; writes history to `CONSULTOR_IA_HISTORICO`

**Other backends:**

- `API.gs` — `getDadosLight()` (fast initial load: COCKPIT + CONFIG) and `getAbasPesadas()` (background load of all 20+ sheets), plus all `google.script.run`-callable functions
- `OpLabExplorer_API.gs` — direct OPLab API passthrough for the explorer panel

### Frontend Layer (`.html` files — Vue 3, no build step)

`Index.html` is the shell. It loads all dependencies (Tailwind CSS CDN, Vue 3 CDN, Plotly, Handsontable) then stitches components together using GAS's `<?!= include('ComponentName'); ?>` template syntax. **AppCore.html is included last** because it bootstraps the Vue app.

**Core frontend files (load order matters):**

| File | Role |
|---|---|
| `LayoutConfig.html` | Defines `window.APP_MENU` (sidebar nav) and `window.LAYOUT_MAP` (which components render per route) |
| `MenuSidebar.html` | Sidebar navigation Vue component |
| `Tradutor.html` | `window.Tradutor`: maps sheet column names (UPPER_SNAKE_CASE) → camelCase JS objects with type coercion (número, moeda, percentual, data, texto). **The COCKPIT sheet has its header at row 10** (rows 1–9 are the portfolio summary panel). |
| `Agregador.html` | `window.Agregador`: business logic / OLAP cube built from cockpit data |
| `Formatador.html` | Display formatting utilities (currency, percentage, date in pt-BR) |
| `AppCore.html` | Vue `createApp` setup: global reactive state (`db` shallowRef), two-phase data loading (light → heavy), client-side routing via `currentView` ref, `provide()` to inject `db` and helpers into all child components |

**Component naming convention:** HTML filename → kebab-case Vue component name (e.g., `CardRadar.html` → `<card-radar>`).

> **Theming:** the app ships a single design with `light`/`dark` modes managed by `window.ThemeManager` in `Styles.html` (`[data-theme="dark"]` token block). An optional "Premium Glass" theme existed previously but was fully removed; recover it via `git revert` of the removal commit if ever needed.

### Data Flow

```
OPLab API → .gs sync engines → Google Sheets
Google Sheets → API.gs (getDadosLight / getAbasPesadas)
  → google.script.run → AppCore.html
  → Tradutor.html (type-safe mapping)
  → db shallowRef (single source of truth)
  → Vue components via provide/inject
```

## Key Conventions

### Sheet References

Always use `SYS_CONFIG.SHEETS.*` constants — never raw string sheet names in `.gs` files:
```js
// Correct
const sheet = ss.getSheetByName(SYS_CONFIG.SHEETS.COCKPIT);
// Wrong
const sheet = ss.getSheetByName("COCKPIT");
```

Sheet names use `UPPER_SNAKE_CASE`. JSON keys for the frontend use `camelCase`. The mapping lives in `SYS_CONFIG.DUD` (backend) and `window.Tradutor.DICIONARIO` (frontend).

### Menu Functions Pattern

All spreadsheet menu actions must go through `_menuBridge()` in `005_CoreServiceUI.gs`:
```js
function MyFeature_Menu() { _menuBridge("My Feature", myFeatureFunction); }
```
`UIHandler.notify()` and `UIHandler.alert()` are silenced (they only log to console). Do not add `SpreadsheetApp.getUi().alert()` calls anywhere — the system operates fully in background mode.

### Logging

Use `SysLogger` everywhere in `.gs` files. It buffers to RAM and batch-writes to the LOGS sheet:
```js
SysLogger.log("ServiceName", "INFO|SUCESSO|AVISO|ERRO|CRITICO", "message", contextObj);
// At the end of each function:
SysLogger.flush();
```
`CRITICO` level triggers an immediate flush. Never write directly to the sheet for logging purposes.

### Standing Rule (user mandate)

**Never remove or alter existing functionality — only add.** ("está proibido retirar ou alterar qualquer funcionalidade atual, apenas acrescentar"). To disable something, comment it out (e.g. the `CardNotional` include in `Index.html` uses `<? /* include('CardNotional'); */ ?>`) rather than deleting it.

### GAS-Specific Rules

- **GAS template comments:** inside `Index.html` scriptlets, use `<? /* ... */ ?>`. The form `<?!/* ... */?>` is invalid and breaks the GAS template compiler with a SyntaxError.

- **Always use `getDisplayValues()`** (not `getValues()`) when reading data that will be sent to the frontend via `google.script.run`. GAS cannot serialize native `Date` objects across that boundary — they arrive as `null`. Type conversion is handled by `Tradutor.html`.
- **Batch all Sheets I/O**: read once, process in memory, write once. Never call `getRange().setValue()` inside a loop.
- `DataUtils.getColMap(aba)` returns `{ HEADER_NAME: zeroBasedIndex }` — use it instead of `indexOf()` loops.
- `DataUtils.getDynamicMap(aba, pkLabel)` returns `{ pkValue: rowObject }` for O(1) lookups.
- `Sanitizador.numeroPuro()` handles BRL currency strings (`"R$ 1.500,50"` → `1500.5`). Use it for all broker-sourced numeric data.

### Black-Scholes Math

`OptionMath` in `013_CoreCalcGreeks.gs` is the canonical math engine. It uses 252 trading days/year (`DIAS_ANO`). `estimateIV()` uses Newton-Raphson with 50 iterations. Do not duplicate this logic elsewhere.

### AI Integration

- **Claude** (`025_ConsultorIAClaudeSonnet45.gs`): Model is `claude-sonnet-4-5`, max tokens `2000`. Analysis results are persisted to `CONSULTOR_IA_HISTORICO` sheet.
- AI personas and prompt templates are configurable per-user via the `Config_Global` sheet (keys: `IA_PERFIL_CONSULTOR`, `PROMPT_REGRAS_GERAIS`, `PROMPT_SISTEMA_*`).

### Adding a New View/Page

1. Create `MyPage.html` with a Vue component registered as `app.component('my-page', { ... })`
2. Add `<?!= include('MyPage'); ?>` to `Index.html` (before `AppCore.html`)
3. Add an entry to `window.APP_MENU` in `LayoutConfig.html`
4. Add an entry to `window.LAYOUT_MAP` in `LayoutConfig.html`
5. Add any new sheet data keys to `window.Tradutor.DICIONARIO` in `Tradutor.html`

### Adding a New Sync Engine

1. Create `0XX_SyncMyFeature.gs` (pick the next number)
2. Expose a top-level function (e.g., `orquestrarSyncMyFeature()`)
3. Add a `_menuBridge` wrapper to `005_CoreServiceUI.gs`
4. Add a menu item in `Código.gs` `onOpen()`
5. Register in `CoreOrchestrator.REGISTRY` in `006_CoreOrchestrator.gs`
6. Add the new sheet to `SYS_CONFIG.SHEETS` in `001_CoreServiceConfig.gs`
7. Add the sheet to `getAbasPesadas()` in `API.gs`
8. Add the column mappings to `window.Tradutor.DICIONARIO` in `Tradutor.html`

---
> Source: [brunotrolo/ControleDeOpcoes](https://github.com/brunotrolo/ControleDeOpcoes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
