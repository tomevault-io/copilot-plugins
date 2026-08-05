## evil-eye-series-n

> > **STATUS: LIVE + FROZEN.** This app is the daily driver — it runs Ryan's

# Evil Eye V1 — session contract

> **STATUS: LIVE + FROZEN.** This app is the daily driver — it runs Ryan's
> real scans, alerts, and scheduler against real API credits. It receives
> **critical fixes and docs changes only**. All feature development happens in
> the V2 rebuild at `../evil-eye-v2` (trunk `main`). If you were asked to
> build something new, you are probably in the wrong folder.

Sports-betting arbitrage finder: scan live odds via The Odds API → show
guaranteed-profit stake splits. Information tool only — it never places bets
or touches bookmaker accounts.

## Commands

```bash
npm test               # Vitest, server AND client workspaces (run from repo root)
npm run typecheck      # tsc for server AND client
npm run dev:server     # Express on :8787 (mock mode without ODDS_API_KEY)
npm run dev:client     # Vite on :5173
```

Verify a backend change end-to-end without spending API credits:

```bash
PORT=8788 ODDS_API_KEY=mock npx tsx server/src/index.ts &
curl -X POST localhost:8788/api/scan -H 'content-type: application/json' \
     -d '{"topN":5,"regionTab":"ca_us"}'
```

## Workflow

- Remote: `git@github.com:RyanParissay/Evil-Eye-Series-N.git` — branch
  `main` = this repo (V1 stable), `v2-dev` = the V2 trunk.
- Work directly on `main` for fixes (small, frozen repo). If isolation is
  needed, worktrees go in `~/evil-eye-worktrees/` on branches `wt/<name>`,
  removed after merge — never scattered elsewhere.
- `server/data/` and `.env` are gitignored runtime state — never commit them.

## Invariants — one-liners (full rules + rationale in ARCHITECTURE.md)

- The Odds API key and Twilio credentials never leave the server process.
- `engine/` stays pure: no Express, fetch, fs, or env.
- Bookmaker filtering happens BEFORE arb detection.
- Line groups are sacred — outcomes only combine within the same |point| group.
- Credits are real money — anything adding calls/regions must update usage math.
- All wall-clock scheduling lives in `server/src/scheduler/`; the only real
  setTimeout is `realTimer.ts`.
- Quiet hours are absolute: zero API calls 01:00–08:00 America/Vancouver.
- Suspicious/same-book arbs are flagged, never hidden — but never pushed.
- Alert dispatch is fire-and-forget; a Twilio outage never fails a scan.
- Nothing is acted on before a record reaches `confirmed` (two-scan pair).
- The safety gate sits AFTER confirmation, in exactly two consumers.
- YELLOW means speculative/simulated only; red = guaranteed arb, green = live.
- Scans drive everything time-based; nothing grows a timer of its own.

## Routing — read before you touch

- **Any server change:** ARCHITECTURE.md "Layering" for the module map, plus
  the matching module paragraph.
- **Scan/engine/odds math:** ARCHITECTURE.md invariants on line groups,
  credits, purity — then `engine/arbitrage.ts`.
- **Adding an option/market/provider/region/channel:** ARCHITECTURE.md
  "Extension recipes" — each is a 2-3 step recipe; don't improvise.
- **Anything surprising (401s, dedup, snapshots, mutexes, MODEL labels):**
  ARCHITECTURE.md "Gotchas" — most "bugs" here are documented decisions.
- **Grading/portfolio rules:** docs/GRADING_RULES.md is binding.

---
> Source: [RyanParissay/Evil-Eye-Series-N](https://github.com/RyanParissay/Evil-Eye-Series-N) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
