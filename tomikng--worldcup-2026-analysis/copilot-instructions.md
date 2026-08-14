## worldcup-2026-analysis

> An automated agent team that analyzes every 2026 FIFA World Cup match (June 11 – July 19, 2026, 104 matches), produces per-match analysis tickets, composes daily betting-ticket recommendations with **virtual stakes only**, and tracks its own performance.

# World Cup 2026 AI Analysis Team

An automated agent team that analyzes every 2026 FIFA World Cup match (June 11 – July 19, 2026, 104 matches), produces per-match analysis tickets, composes daily betting-ticket recommendations with **virtual stakes only**, and tracks its own performance.

## Hard guardrails

- **Never place, automate, or facilitate real-money bets.** All stakes are virtual units against a simulated bankroll. Recommendations are analysis, not financial advice.
- **Never invent data.** Every odds value must come from a fetched source recorded in `sources`. If odds can't be found for a market, omit that market.
- **The ledger is append-only truth.** Never hand-edit `data/ledger.json`; it is rebuilt by `scripts/settle.py`.
- Timestamps are always UTC ISO-8601 (`2026-06-11T19:00:00Z`). A **matchday** `YYYY-MM-DD` covers kickoffs from 06:00 UTC that day to 05:59 UTC the next day — North-America evening games that cross UTC midnight belong to the previous (local) matchday.

## Pipeline (what /daily-run does)

1. Settle yesterday: `python3 scripts/fetch_results.py <yesterday>`, stats enrichment (see /settle skill), then `python3 scripts/settle.py`
2. Fetch today: `python3 scripts/fetch_fixtures.py <today>`
3. Polymarket prices: `python3 scripts/fetch_polymarket.py <today>` and `python3 scripts/fetch_polymarket.py --futures`
4. `odds-researcher` agent → `data/odds/<today>.json`
5. One `match-analyst` agent per fixture (parallel) → `data/tickets/<today>/<match_id>.json`
6. `ticket-builder` agent → `data/betslips/<today>.json`
7. Futures review (agent step in /daily-run) → may add positions to `data/futures.json`
8. `python3 scripts/build_index.py` (regenerates `data/index.json` for the dashboard)
9. Commit (`daily: YYYY-MM-DD`) and push — the dashboard on GitHub Pages updates on push.

## Data sources (free tier)

- **Fixtures/results:** football-data.org v4, competition code `WC`, header `X-Auth-Token` from `FOOTBALL_DATA_TOKEN` (in gitignored `.env`). 10 req/min. If the token is missing, the scripts exit with code 2 and agents fall back to web research (fifa.com, bbc.com/sport) writing the identical schema.
- **Odds:** no free bookmaker odds API covers the World Cup — the `odds-researcher` agent web-researches public odds pages (oddschecker.com, covers.com, oddsportal.com, bookmaker public sites) and records the sources used.
- **Polymarket:** free public Gamma API (`gamma-api.polymarket.com`, no key) via `scripts/fetch_polymarket.py` — per-match prices and futures (tournament/group winners). Prices are probabilities (0–1); decimal-odds equivalent = 1/price. Polymarket uses FIFA team naming ("Korea Republic"); aliases live in `fetch_polymarket.py`. Tomas places his real bets on Polymarket, so PM recommendations must be actionable: outcome, limit price ("buy ≤ 43¢"), and the market URL.

## JSON schemas (data/)

All files written by scripts or agents MUST match these shapes exactly.

### fixtures/YYYY-MM-DD.json
```json
{"date": "2026-06-11", "source": "football-data.org", "fetched_at": "<iso>",
 "matches": [{"match_id": "fd-12345", "kickoff_utc": "<iso>", "stage": "GROUP_STAGE",
              "group": "GROUP_A", "home": "Mexico", "away": "South Africa", "venue": "Estadio Azteca"}]}
```
`match_id` is `fd-<football-data id>`; web-research fallback uses `wc-<seq>` slugs but must stay stable across files.

### results/YYYY-MM-DD.json
```json
{"date": "2026-06-11", "source": "football-data.org", "fetched_at": "<iso>",
 "matches": [{"match_id": "fd-12345", "home": "Mexico", "away": "South Africa",
              "home_goals": 2, "away_goals": 1, "status": "FINISHED",
              "home_cards": 2, "away_cards": 3, "home_reds": 0, "away_reds": 1,
              "home_corners": 7, "away_corners": 3,
              "stats_sources": ["https://..."]}]}
```
`status`: `FINISHED` | `POSTPONED` | `CANCELLED`. Goals are full-time (after extra time in knockouts; note shootout in `notes` if any — h2h market in knockout stage settles on the result after ET, draw possible only in group stage).

The six stats fields (cards = yellows+reds per team, reds, corners) are **enriched by the /settle agent from web-researched match reports** (FIFA/BBC/ESPN) — football-data.org free tier doesn't provide them. Set a field to `null` whenever it can't be verified from a real source; `settle.py` then **voids** any leg depending on it. Never guess stats.

### odds/YYYY-MM-DD.json
```json
{"date": "2026-06-11", "collected_at": "<iso>",
 "matches": [{"match_id": "fd-12345",
   "h2h": {"home": 2.10, "draw": 3.30, "away": 3.60},
   "double_chance": {"1x": 1.28, "12": 1.32, "x2": 1.72},
   "dnb": {"home": 1.55, "away": 2.45},
   "totals": {"line": 2.5, "over": 1.95, "under": 1.85},
   "totals_alt": [{"line": 1.5, "over": 1.45, "under": 2.70},
                  {"line": 3.5, "over": 3.10, "under": 1.36}],
   "team_totals": {"home": {"line": 1.5, "over": 2.00, "under": 1.80},
                   "away": {"line": 0.5, "over": 1.65, "under": 2.20}},
   "btts": {"yes": 2.05, "no": 1.75},
   "cards": {"line": 4.5, "over": 1.90, "under": 1.90},
   "red_card": {"yes": 3.50, "no": 1.30},
   "corners": {"line": 9.5, "over": 1.90, "under": 1.90},
   "correct_score": {"1-0": 6.00, "2-1": 8.50, "2-0": 7.00},
   "sources": ["https://..."]}]}
```
Decimal odds. Every market is optional — omit any market not found; never guess. `correct_score` lists only the 3–5 most likely scores found.

### polymarket/YYYY-MM-DD.json and polymarket/futures.json
Written by `scripts/fetch_polymarket.py` — per-match events with `markets` (question, outcomes, prices 0–1, event url), and a futures snapshot (tournament/group winners, top outcomes by price). Read-only inputs for agents; never hand-edit.

### tickets/YYYY-MM-DD/<match_id>.json (analysis ticket)
```json
{"match_id": "fd-12345", "date": "2026-06-11", "home": "Mexico", "away": "South Africa",
 "kickoff_utc": "<iso>", "stage": "GROUP_STAGE", "group": "GROUP_A", "venue": "...",
 "analysis": {"summary": "2-4 sentences", "home_form": "...", "away_form": "...",
              "h2h": "...", "injuries_news": ["..."], "key_factors": ["..."]},
 "prediction": {"probs": {"home": 0.45, "draw": 0.28, "away": 0.27},
                "predicted_score": "2-1", "confidence": 0.65},
 "market_probs": {"totals_2.5_over": 0.55, "btts_yes": 0.52,
                  "cards_4.5_over": 0.48, "corners_9.5_over": 0.50},
 "value_notes": "where model probs diverge from market implied probs",
 "sources": ["https://..."], "generated_at": "<iso>"}
```
`probs` must sum to 1.0 (±0.01). `confidence` ∈ [0,1]. `market_probs` is optional and holds explicit probabilities for extra markets, keyed `<market>_<line>_<selection>` (or `<market>_<selection>` for btts/red_card) — include ONLY markets the research genuinely supports (e.g. cards via referee discipline record + teams' card averages; corners via attacking style). The ticket-builder may only bet markets present here or in `prediction.probs`.

### betslips/YYYY-MM-DD.json
```json
{"date": "2026-06-11",
 "slips": [{"slip_id": "2026-06-11-S1", "type": "single", "stake": 10.0,
   "legs": [{"match_id": "fd-12345", "market": "h2h", "selection": "home",
             "line": null, "odds": 2.10, "model_prob": 0.52, "edge": 0.094,
             "rationale": "one sentence"}],
   "combined_odds": 2.10, "status": "pending", "payout": null, "settled_at": null,
   "rationale": "one sentence for the slip"}]}
```
- `market` and selections:
  - `h2h`: `home|draw|away` · `double_chance`: `1x|12|x2` · `dnb`: `home|away`
  - `totals`: `over|under` + `line` · `team_totals`: `over|under` + `line` + `side` (`home|away`)
  - `btts`: `yes|no` · `correct_score`: `"2-1"` (home-away)
  - `cards`: `over|under` + `line` · `red_card`: `yes|no` · `corners`: `over|under` + `line` (stat markets — void if stats unverified)
- Optional leg fields: `side` (team_totals only), `exchange` (`book` default | `polymarket`), `pm` (`{"price": 0.43, "url": "https://polymarket.com/event/..."}`, required when exchange is polymarket; leg `odds` = round(1/price, 2); rationale phrased as an order, e.g. "buy Yes ≤ 43¢").
- `type`: `single` (1 leg) or `accumulator` (2–5 legs). Slip ids: `<date>-S<n>` singles, `<date>-ACC` accumulator.
- `combined_odds` = product of leg odds. `status`: `pending|won|lost|void` — only `scripts/settle.py` changes it.

### Staking policy (ticket-builder)
- Bankroll starts at 1000 units. Recommend only legs with positive edge ≥ 0.05: `model_prob − 1/odds` for book legs, `model_prob − price` for Polymarket legs.
- Line-shop: when book and Polymarket price the same outcome, use whichever gives the bigger edge.
- Single stakes: 5 units (edge 0.05–0.08), 10 (0.08–0.15), 15 (0.15–0.25), 20 (>0.25). Accumulator: flat 5 units, max one per day, 2–4 legs.
- Max total daily stake: 60 units. Max one slip per market per match; never both sides. Zero qualifying edges → write the file with `"slips": []`; never force a bet.

### Futures (data/futures.json)
```json
{"budget": 100.0, "positions": [
  {"position_id": "FUT-1", "market_title": "World Cup Group A Winner",
   "outcome": "Mexico", "entry_price": 0.565, "stake": 10.0,
   "url": "https://polymarket.com/event/world-cup-group-a-winner",
   "opened": "2026-06-11", "status": "open", "resolved_at": null,
   "rationale": "one sentence"}]}
```
- Separate tournament-long budget of 100 units (NOT part of the 60u daily cap). Open a position only when model probability − price ≥ 0.05 and remaining budget allows; stakes 5–15u. Buy-and-hold to resolution — no early exits (the futures review may *note* "would exit here" in its summary, but the ledger holds).
- `status` (`open|won|lost`) is set by the /settle agent only after verifying official resolution (group winners after the group stage ends June 27; tournament winner July 19). `settle.py` does the math: payout = stake / entry_price on won.

### ledger.json
Rebuilt from scratch by `settle.py` from all betslip files (idempotent). Shape:
```json
{"starting_bankroll": 1000.0, "bankroll": 987.5, "updated_at": "<iso>",
 "stats": {"total_staked": 0, "total_returned": 0, "profit": 0, "roi": 0,
           "slips_won": 0, "slips_lost": 0, "slips_void": 0, "slips_pending": 0,
           "hit_rate": 0, "by_market": {"h2h": {"staked": 0, "returned": 0, "won": 0, "lost": 0}}},
 "history": [{"date": "2026-06-11", "bankroll": 990.0}]}
```

## Settlement rules (settle.py is the only authority)

- h2h: group stage = 90-min result; knockout = result after extra time (draw impossible).
- double_chance: `1x` covers home+draw, `12` home+away, `x2` draw+away.
- dnb: draw → void (stake back); otherwise as h2h.
- totals / team_totals / cards / corners: value vs `line` (over wins if value > line; exactly on a .0 line = void push). team_totals uses the `side` team's goals; cards = both teams' yellows+reds; corners = both teams'.
- correct_score: exact full-time score `"H-A"`.
- btts: both teams scored ≥1. red_card: any red card in the match.
- **Stat markets void when stats are null** in the results file (unverified match report).
- `POSTPONED`/`CANCELLED` match → leg void. Void single → stake returned. Void leg in accumulator → that leg's odds become 1.0. Any lost leg → slip lost, payout 0. All legs won/void → payout = stake × adjusted combined odds.
- Bankroll = starting − total settled stakes + total payouts. Pending stakes are deducted (committed) the day placed.

## Conventions

- Python 3.14, stdlib only (urllib, json, argparse) — no pip dependencies; tests use pytest.
- Scripts are idempotent and safe to re-run; exit 0 success, 1 error, 2 missing-token.
- Run tests: `python3 -m pytest tests/ -q`.
- Dashboard is static (repo root `index.html` + `assets/`), served by GitHub Pages, reads `data/*.json` via relative fetch using `data/index.json` as the directory of available dates.

---
> Source: [tomikng/worldcup-2026-analysis](https://github.com/tomikng/worldcup-2026-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
