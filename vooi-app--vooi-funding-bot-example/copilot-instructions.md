## vooi-funding-bot-example

> This file is a contract between humans and any AI agent (Claude, Cursor, Aider, etc.) that touches this repo. Read it before making changes.

# AGENTS.md — guidance for AI assistants working in this repository

This file is a contract between humans and any AI agent (Claude, Cursor, Aider, etc.) that touches this repo. Read it before making changes.

## What this project is

`vooi-funding-arb-bot` is a Python bot that runs delta-neutral funding-rate arbitrage on the VOOI Perps API (Hyperliquid + Lighter, optionally Aster). It places real orders against real money. Every change must be reviewed with that in mind.

## Hard rules

1. **Never commit secrets.** `.env` is gitignored. If you see a bearer token or any credential in a file other than `.env`, treat it as a leak — flag it and refuse to commit.
2. **Never edit `.env`.** Only `.env.example` (a template). Real values are operator-managed.
3. **Never widen access without explicit confirmation.** Do not lower `BOT_STOP_LOSS_PCT`, raise position sizes, or remove safety checks unless the user asked for that specific change in that specific cycle.
4. **`BOT_DRY_RUN=true` is the safe default.** Do not flip it to `false` in `.env.example` or any code path. Operators flip it themselves.
5. **Never auto-trade in tests.** Tests that hit live APIs must be marked with `@pytest.mark.integration` and require an explicit opt-in (`pytest -m integration`).
6. **Reproducible builds.** Do not bump `pyproject.toml` dependencies without re-locking `uv.lock` and running the full test suite.

## Repository layout

```
fundbot/         main bot package (mvp.py is the engine)
probe/           Phase-0 API validation tool (read-only and tiny write probes)
scripts/         operational helpers (close_one, recover_snapshot, etc.)
tests/           pytest suite (unit + a few integration tests behind a marker)
skills/          Claude Code skills (e.g. funding-arb-cycle MCP co-pilot)
docs/            STRATEGY.md and a few targeted explainers
.env.example     template — copy to .env to run the bot
```

## Strategy quick reference

The bot ranks opportunities by `netApr`, opens delta-neutral pairs that pass a filter chain, and closes by a priority list (`hard_stop_loss` → `max_hold` → `min_hold gate` → `smart_neg_N` → `smart_decl_N` → `low_apr_N`). All knobs live in `.env.example`. See `docs/STRATEGY.md` for the full algorithm.

## How to make a code change

1. Read `docs/STRATEGY.md` first if the change touches strategy logic.
2. Search for callers and existing tests before editing. Most edits to `fundbot/mvp.py` should come with a test in `tests/`.
3. Run `uv run ruff check .` and `uv run mypy fundbot probe` before requesting review.
4. Run `uv run pytest -q` (excludes integration tests by default).
5. For strategy changes, run a 24h dry run (`BOT_DRY_RUN=true`) and inspect `state.ndjson` for surprising decisions before going live.
6. Commit with a short clear message. Avoid mass renames unrelated to your change.

## How to deploy

See `docs/DEPLOY.md` for a generic Fly.io setup. For other orchestrators, the bot only needs a writable volume (state files), one process, and the `.env` file mounted.

## What to surface to the human, always

- Any change that increases position size, leverage, or per-exchange margin cap.
- Any change to the close-criteria priority list or thresholds.
- Any new outbound call (new venue, new API endpoint).
- Any new dependency.

## Things AI agents commonly get wrong

- **Don't auto-retry failed writes.** `POST /exchange/orders` is not idempotent on every venue — a retried 5xx can double-open. The bot's existing logic distinguishes safe vs unsafe retries; preserve it.
- **Lighter ignores `clientOrderId` when bracket SL/TP are attached.** The bot has a fallback that matches by `(asset, side, size)`. Don't "fix" the apparent missing coid — it's expected.
- **SSE is additive, not a replacement.** `fundbot/sse.py` lets pollers wake on push events, but every decision is still REST-confirmed. Don't try to skip the REST check after an SSE wake — the matcher is generous on purpose (spurious wakes are cheaper than missed events). If you find yourself wanting to trust the SSE payload directly, you almost certainly want to extend the REST check instead.
- **Hyperliquid clientOrderId must be `0x` + 32 hex chars.** Other formats are rejected.
- **`fundingFee` is `null` on Aster.** Code that aggregates funding must handle None.
- **Don't fetch all positions in a tight loop.** The bot's reconcile is rate-aware — respect it.
- **Don't treat a partial fill as a full fill.** When a LIMIT order is cancelled with a position present, verify size against `matched_size` before continuing the cycle.

## Style

- Type hints on every public function.
- Decimal, not float, for prices and sizes.
- Structured logging via `NDJsonLog.emit` — one event per significant action. Don't add free-form `print()`.
- Keep `mvp.py` the single source of strategy truth for now; the package directories under `fundbot/` (api/, strategy/, etc.) are stubs reserved for a future refactor.

## When in doubt — stop and ask the human

This bot can lose money fast. There is no shame in asking before acting.

---
> Source: [vooi-app/vooi-funding-bot-example](https://github.com/vooi-app/vooi-funding-bot-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
