## crypto-trading-bot-ai

> Project Intelligence (.cursorrules)

Project Intelligence (.cursorrules)

Workstyle & preferences
- Iterate in small, testable batches; allow user to validate between steps.

Critical patterns
- Env vars: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `VITE_ALPACA_API_KEY`, `VITE_ALPACA_SECRET_KEY`, `VITE_GROQ_API_KEY`, optional `VITE_COINGECKO_API_KEY`.
- Fallback-first services: CoinGecko and Alpaca return structured fallback data on failure; never hard-crash UI.
- Groq rate limit: enforce 3s min interval; parse JSON from model output by regex; provide fallback insights.
- RLS: Keep authenticated-only policies; avoid reintroducing anon/demo policies in production.
- Layout: `DraggableGrid` saves layouts in `localStorage`; resolves overlaps on load and after drag/resize.
- API health: Header chip reflects summarized status; modal performs live probes.

Integration edges
- Orders placed via Alpaca are not automatically mirrored into Supabase `orders` (portfolioService path exists but is unused by UI). Decide on a single source of truth or implement bridging.
- Price sources: App uses CoinGecko for `cryptoData`; Alpaca also has a crypto bars helper. Prefer one canonical source per view to avoid discrepancies.

UX conventions
- Keep loading skeletons and clear empty states.
- Prefer showing partial data over blocking views.

---
> Source: [kakha012/crypto-trading-bot-ai](https://github.com/kakha012/crypto-trading-bot-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
