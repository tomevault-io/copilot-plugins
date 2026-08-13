## coinlobster-gemini-extension

> CoinLobster gives you live crypto whale data across 15 centralized exchanges,

# CoinLobster

CoinLobster gives you live crypto whale data across 15 centralized exchanges,
Hyperliquid (a perp venue whose public feed names the wallet on both sides of a
fill; counted separately from the 15, never folded into them) and on-chain DEX
swaps on Ethereum, Base and Arbitrum. All tools are read-only market data:
describe what the data shows, never give financial advice or forecasts.

## Start here

- `whale_radar`: which coins have unusual whale flow right now (1h, 4h, 24h
  windows). The best first call for "what's going on in crypto".
- `whale_trades`: live whale trades merged across all venues, newest first.
  Filter by coin, source (cex/dex/all), or min_usd. `whale_tape` is a deprecated
  alias of this tool, kept for old clients; prefer `whale_trades`.
- `market_overview`: one-call market-wide scan (totals, 24h whale net flow, BTC
  risk regime).
- `trade_context`: the one-call picture for a single pair.

## The rest, by question

- One coin's flow: `whale_flow` (hourly buy vs sell, CEX and DEX split),
  `whale_context` (is this flow unusual for THIS coin).
- Track record: `flag_outcomes`, signals that fired with fire-time price and
  what price did after. Use it to answer "did this matter last time".
- One pair's state: `market_snapshot` (price, 24h stats, average funding, open
  interest, recent liquidation totals), `funding_matrix` (per-venue funding),
  `squeeze_score` (crowding; needs a developer plan).
- Forced closes: `liquidations` (one pair), `market_liquidations` (market-wide),
  `liq_zones` (projected levels, a model, not executed trades),
  `defi_liquidations` (lending liquidations, a DIFFERENT event from a perp
  liquidation and never added to one).
- On-chain and named wallets: `onchain_whales`, `dex_wallet`, `defi_summary`,
  `hl_whales`, `hl_board`, `hl_wallet`.
- Recorded history: `market_history` (the daily archive), `market_extremes`
  (records and top/bottom 5% against a pair's own history).
- Screening: `market_screener`. News: `crypto_news` (headlines plus a sentiment
  score, -100 to +100, for a coin and the macro picture).

## Usage notes

- Call `tools/list` rather than assuming a tool exists. The list above is a map,
  not the catalogue: the server is the source of truth and every tool ships its
  own description.
- Keyless calls get the free data shape: BTC/USD live in full, every $1M+ CEX
  trade on any coin live, other pairs delayed 15 minutes, DEX swaps $250K or
  larger. A free API key does NOT change this: full MCP depth needs an active
  developer plan key (Builder $29/mo, Growth $99, Scale $249). CoinLobster Pro
  ($9/mo) is the website subscription and grants no MCP depth or API access.
  Plans: https://coinlobster.com/developers
- Cost: on an active developer plan each successful tool call spends 40 to 100
  credits (2x the mirrored REST cost) from the plan's shared balance. Keyless
  calls, discovery and failed calls cost nothing.
- Amounts are USD. `side: buy` means an aggressive buyer lifted the offer.
- When a response includes a `depth_note`, respect it when describing history
  depth. Every number carries the window it was measured over; keep that window
  attached when you quote it.

---
> Source: [CoinLobster/coinlobster-gemini-extension](https://github.com/CoinLobster/coinlobster-gemini-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
