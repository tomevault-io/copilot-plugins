## url-to-markdown-cli-tool

> - Node.js CLI that converts web pages into clean, LLM-friendly markdown.

# AGENTS.md

Purpose
- Node.js CLI that converts web pages into clean, LLM-friendly markdown.
- Fetches pages via Puppeteer, strips noise, and converts HTML to markdown.
- No external APIs or keys required.

Structure
- `src/index.js`: CLI entry point and option parsing.
- `src/lib/pageFetcher.js`: Puppeteer fetching.
- `src/lib/markdownProcessor.js`: HTML to markdown conversion.
- `tests/`: Jest tests with fixtures in `tests/fixtures/`.

Tech
- Node.js >= 18.
- Core deps: puppeteer, cheerio, turndown, commander.
- Tests: jest (`npm test`).

---
> Source: [mmdclx/url-to-markdown-cli-tool](https://github.com/mmdclx/url-to-markdown-cli-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
