## pi-x-search

> Pi package providing one global `x_search` tool through xAI's Responses API.

# AGENTS.md

Pi package providing one global `x_search` tool through xAI's Responses API.

## Commands

- `npm run check` — TypeScript check.
- `pi -e . --list-models` — verify the package loads without making a model request.
- `npm pack --dry-run` — inspect the npm package contents.

## Design

- Reuse pi's built-in `xai` credential through `ctx.modelRegistry.getApiKeyForProvider("xai")`.
- Do not add another xAI provider, OAuth flow, token store, browser-cookie reader, or X scraper.
- Keep the extension focused on X search; unrelated xAI tools belong elsewhere.

---
> Source: [samirettali/pi-x-search](https://github.com/samirettali/pi-x-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
