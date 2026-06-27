## portalrelatorios

> - **Dados do Empreendimento** (Power BI embed) — these npm scripts exist:

# Agent Instructions

## Scope

- **Dados do Empreendimento** (Power BI embed) — these npm scripts exist:

  **`npm run crawl:dados-empreendimento`** — pure one-pass crawl (headless): `--resume --fast`.

  **`npm run crawl:dados-empreendimento:headed`** — pure one-pass crawl with **`--headed --fast`** so Chromium is visible.

  **`npm run crawl:dados-empreendimento:retry`** — retry loop (headless): `--resume --fast` on each pass; retries after **`CRAWL_RETRY_SECONDS`** (default `90`) until exit 0.

  **`npm run crawl:dados-empreendimento:retry:headed`** — same retry loop with **`--headed --fast`** so Chromium is visible.

  Extra flags after `--`, e.g. **`npm run crawl:dados-empreendimento -- --quiet`**, **`npm run crawl:dados-empreendimento:headed -- --refresh-options`**, or the matching `:retry` scripts.

- **Checkpoint**: `data/resultados-leiloes-geracao/dados-do-empreendimento/crawl-state.json`. Delete it to reset progress (existing `by-empreendimento/*.json` still skipped unless **`--force`**).

- Crawler logic (`by-empreendimento.js`): open Empreendimento slicer → merge visible labels into **`option_names`** → extract first visible row missing JSON → scroll virtual list → reopen-from-top passes when stalled.

- Browser viewport is fixed at **1920×1080**. Initial portal/report load retries with **`page.goto` refresh** on failure (`CRAWL_LOAD_MAX_ATTEMPTS`, default `5`; `CRAWL_LOAD_RETRY_MS`, default `3000`).

- Optional env: `OPTION_DISCOVERY_STALE_SCROLLS`, `OPTION_DISCOVERY_REOPEN_PASSES`, `CRAWL_MIN_EXPECTED_OPTIONS`, `--allow-small-list`.

- Cursor **IDE Browser MCP**: outer ANEEL shell only; Power BI iframe is **Node Playwright** in `by-empreendimento.js`.

- Do not reintroduce a Fastify API unless the user asks.

- Never commit real `.env` secrets.

## Verification

```bash
node --check src/pages/resultados-leiloes-geracao/reports/dados-do-empreendimento/crawlers/by-empreendimento.js
node --check src/pages/resultados-leiloes-geracao/reports/dados-do-empreendimento/parsers/visual-cards-parser.js
```

## Data

- Generated JSON: `data/resultados-leiloes-geracao/dados-do-empreendimento/by-empreendimento/`. Preserve unless regenerating.

---
> Source: [Guilheeeeeeerme/portalrelatorios](https://github.com/Guilheeeeeeerme/portalrelatorios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
