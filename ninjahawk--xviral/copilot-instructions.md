## xviral

> Local replica of X's For You pipeline (built from the official `xai-org/x-algorithm`

# XViral — project context

Local replica of X's For You pipeline (built from the official `xai-org/x-algorithm`
May 2026 release) used as a pre-post scorer for a crypto-Twitter giveaway account
(~7k followers). Read `FINDINGS.md` (source audit), `PLAN.md` (roadmap + accuracy
contract), `simulator/experiments/RESULTS.md` (format evolution log) before changing
anything.

## Ground rules

- **Never add Claude attribution to commits or PRs** — no Co-Authored-By/Generated-with
  trailers, ever. Commits are authored solely by ninjahawk.
- **Judge consistency:** scores are only comparable within a judge configuration.
  Do not change `JUDGE_PROMPT`, `weights.json`, the model, or temperature between
  batches you intend to compare — and note the break in RESULTS.md if you do.
- **README style is deliberate** — 1:1 modeled on `ninjahawk/Subtext` (research-artifact
  voice, centered header, numbered "moments", Method/Validation/Limitations). Keep it.
- **Never commit `x-algorithm/phoenix/artifacts/` contents** (2.9 GB, gitignored);
  re-fetch via the LFS batch API if missing.
- The vendored `x-algorithm/` stays unmodified (Apache-2.0 reference copy).

## Machine facts (verified 2026-07-13)

- Ollama judge engine: `qwen3.5:9b` works — **must pass `"think": false`** with
  `format:"json"` or the response comes back empty. `qwen3.6:35b-a3b` fails to load
  (14 GB host-buffer OOM); `qwen3.6-27b-ctx` from the global CLAUDE.md no longer exists.
- Quality engine: `claude -p` (subprocess). ~5-6 s/draft on the 9B.
- Reading X without API: `https://cdn.syndication.twimg.com/tweet-result?id=<ID>&token=a`
  → full tweet JSON with engagement counts, no auth (this is the calibration data path).
  Direct x.com fetches return HTTP 402.

## Current state / next step

Format evolution converged (23 variants, 3 generations): champion family =
**video payout-proof + no-strings address ask** (see RESULTS.md for exact texts and
the 5-point recipe). Owner reports sim rankings match real account behavior
(directional validation). Phase 2 (calibration harness: prediction log, outcomes
fetcher, pairwise-accuracy report; target ≥70% out-of-sample) is designed in PLAN.md
and **blocked on the account handle + benchmark accounts**. First real-world action:
post champion vs runner-up as calibration pair #1.

---
> Source: [ninjahawk/XViral](https://github.com/ninjahawk/XViral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
