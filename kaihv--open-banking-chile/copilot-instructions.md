## open-banking-chile

> Multi-bank scraping framework for Chilean banks. Uses Puppeteer + headless Chrome to extract bank movements and balances. Plugin architecture: each bank is a separate module implementing `BankScraper` interface.

# Open Banking Chile

## Context for Gemini
Multi-bank scraping framework for Chilean banks. Uses Puppeteer + headless Chrome to extract bank movements and balances. Plugin architecture: each bank is a separate module implementing `BankScraper` interface.

## Quick start
1. Install Node.js 18+ and Google Chrome
2. `npm install && npm run build`
3. Set env vars: `FALABELLA_RUT`, `FALABELLA_PASS`
4. Run: `node dist/cli.js --bank falabella --pretty`

## Key files
- `src/banks/falabella.ts` — Banco Falabella scraper
- `src/types.ts` — TypeScript interfaces (BankScraper, BankMovement, ScrapeResult)
- `src/utils.ts` — Shared utilities (formatRut, findChrome, closePopups)
- `src/index.ts` — Bank registry (getBank, listBanks)
- `src/cli.ts` — CLI (--bank, --list, --movements, --screenshots)

## Adding a new bank
Create `src/banks/<id>.ts` exporting a `BankScraper` object, register in `src/index.ts`. See CONTRIBUTING.md.

## Security
- All local, no external servers
- Credentials via env vars only
- Screenshots may contain sensitive data

---
> Source: [kaihv/open-banking-chile](https://github.com/kaihv/open-banking-chile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
