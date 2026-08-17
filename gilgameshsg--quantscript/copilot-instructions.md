## quantscript

> Orientation for AI coding agents working in the QuantScript repo. `README.md` is written for humans; this file is written for agents and is loaded with every prompt.

# AGENTS.md

Orientation for AI coding agents working in the QuantScript repo. `README.md` is written for humans; this file is written for agents and is loaded with every prompt.

## Project overview

QuantScript is an open-source trading SDK owned by **Gilgamesh Pte. Ltd.** (Singapore). It is a domain-specific language (`.qs`, JavaScript-compatible), a backtesting engine, and a simulated broker. Strategies are authored in QuantScript and backtested offline against monthly parquet candle data. There is no live trading in this repo.

## Repository layout

| Path | Contents |
|---|---|
| `packages/lang/` | Language core — lexer, parser, interpreter, backtest engine (CLI: `node packages/lang/dist/cli/index.js run1`) |
| `packages/ta/` | Technical indicators (`sma`, `ema`, `rsi`, `atr`, `adx`, `supertrend`, ...) |
| `packages/data/` | Candle adapters — parquet and CSV loading |
| `packages/journal/` | Structured JSONL run journal |
| `packages/analytics/` | Trading performance analytics |
| `packages/sim-broker/` | Simulated broker (`SimEngine`/`SimBroker`) |
| `mybacktests/qs/` | Strategy sources (`.qs`) and generated `.mjs` ports |
| `mybacktests/ohlc/` | Monthly parquet candle data `{SYMBOL}_{TF}_{YYYY-MM}.parquet` |
| `mybacktests/tools/` | `qsto.mjs` (converter), `parse-results.mjs` (report), `csv-to-parquet.mjs` |

## Build and test

- Node.js >= 22 (see `.nvmrc`)
- `npm install`
- `npm run build` — builds packages in dependency order (data → journal → analytics → ta → sim-broker → lang)
- `npm test` — runs the vitest suite for every package
- Single package: `npx vitest run --root packages/<name>`

## Conventions and invariants

- **The JSONL journal is the sole source of truth** for backtest results. Never parse console output; never create `.log` files.
- **Never parse journals with PowerShell** — `ConvertFrom-Json` silently coerces ISO 8601 strings to `[System.DateTime]`, breaking string methods. Use `mybacktests/tools/parse-results.mjs`.
- **Never pipe a backtest run through `Select-Object -First N`** — it terminates the pipeline and kills the run mid-way.
- **Never edit `.qs`/`.mjs` for scheduling** — date ranges, data dirs, and symbols are always CLI flags.
- Backtests run synchronously; even multi-year runs complete in seconds.
- Data availability: 15m parquet only — `XAUUSD_15m`, from 2020-05. No 1m fill series and no CSV fallback in this repo.

## Skills

| Skill | Location | When to use |
|---|---|---|
| Backtesting | `.agents/skills/backtest/SKILL.md` | Asked to backtest a strategy, test a date range, compare cost models, or validate a strategy — both the CLI and `.mjs` pipelines, flag reference, and report expectations |

---
> Source: [GilgameshSG/quantscript](https://github.com/GilgameshSG/quantscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
