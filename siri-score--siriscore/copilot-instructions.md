## siriscore

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install (editable, with dev deps)
pip install -e ".[dev]"

# Web UI — serves index.html at http://localhost:8000
uvicorn api.main:app --reload

# Offline demo — open directly in a browser, no server needed
open web/index.html   # uses mock data

# CLI
btc-privacy-check --psbt <base64>
btc-privacy-check --rawtx <hex>
btc-privacy-check --txid <txid>
btc-privacy-check --psbt <b64> --import-sparrow sparrow-labels.json
btc-privacy-check --psbt <b64> --fail-below 60   # exits 1 if score < threshold
btc-privacy-check --psbt <b64> --json

# Tests
pytest                                                                          # all
pytest tests/test_labels.py                                                     # single file
pytest tests/test_heuristics.py::TestH2RoundAmount::test_fires_on_round_output # single test
pytest --cov=scorer                                                             # with coverage
```

## Architecture

The project is three layers over a shared `scorer/` library:

```
scorer/          ← standalone Python library (importable by wallets)
  __init__.py    ← public API: score(input_str) → Report, import_labels(path) → int
  report.py      ← data models: Severity enum, Finding dataclass, Report dataclass
  parser.py      ← dispatch: base64 PSBT → _parse_psbt, hex → _parse_rawtx, 64-char hex → txid lookup
                 ←   raw-tx prevout enrichment is gated on lookup/backend — offline stays offline
  lookup.py      ← blockstream.info/mempool.space REST wrapper with in-process dict cache;
                 ←   used by H3, H4, txid fetch, and opt-in raw-tx prevout enrichment
  labels.py      ← SQLite store at ~/.utxo-privacy-scorer/labels.db; Sparrow JSON import
  heuristics/    ← one module per heuristic, all share the same signature:
    __init__.py  ←   check(tx, psbt_meta) → Finding | None
                 ←   ALL list drives the engine in scorer/__init__.py
cli.py           ← argparse + rich; calls scorer.score(), renders coloured output
api/main.py      ← FastAPI; POST /score, GET/POST /labels, POST /labels/import
web/index.html   ← single-file vanilla HTML/CSS/JS frontend (no build step); spec in FRONTEND.md
```

### Scoring engine (scorer/\_\_init\_\_.py)

`score(input_str)` calls `parse()`, then runs every module in `heuristics.ALL` and collects `Finding` objects. Score = `min(100, max(0, 100 - sum(weights)))`. Negative weights (H10: −10) add to the score. If H8 fired (tainted UTXO), the final score is additionally capped at 40.

If H9 or H10 fires, the engine strips H5 from findings before scoring (coinjoin breaks the CIOH assumption — H5 would be a false positive).

### Heuristic contract

Every heuristic is a pure function `check(tx, psbt_meta) → Finding | None`. Adding a new heuristic means: create `scorer/heuristics/hN_name.py`, implement `check()`, append the module to `LOCAL` (or `NETWORK` if it needs outbound calls) in `scorer/heuristics/__init__.py`. Also add an entry to `_HEURISTIC_DEFS` in `scorer/__init__.py`.

**Positive heuristics** (H9, H10, H11) set `positive=True` and use `weight=0` (no effect) or a negative weight (bonus). The engine and UI treat them differently — they render green and appear as `pass` in the checks list, not `fail`.

### H8 override behaviour

H8 (`h8_tainted_label.py`) is special: weight=25 but the engine imposes a hard cap of 40 on the final score when H8 fires. Cap defined as `_H8_SCORE_CAP = 40` in `scorer/__init__.py`.

### H9 / H10 — coinjoin suppression

H9 detects coinjoin-sourced inputs (Whirlpool denominations or `coinjoin` label tag). H10 detects whether the tx being scored is itself a coinjoin (Whirlpool / Wasabi / JoinMarket structure). When either fires, H5 (CIOH) is stripped from findings — it would be a false positive.

### H11 — Payjoin opportunity

H11 reads `payment_uri` or `bip21_uri` from `psbt_meta` and checks for a BIP-21 `pj=` parameter (BIP-77 async Payjoin endpoint). Weight=0. Escalates to WARNING if H5 also fires.

### Label store

`labels.py` manages a local SQLite file. The `tag` field drives scoring: `tainted` triggers H8; `coinjoin` triggers H9 (suppresses H5); `clean` and `unknown` are informational. Import sources: Sparrow BIP329 JSONL, manual via CLI/API.

### Frontend

`web/index.html` is a self-contained demo with hardcoded mock data — no backend required to open it directly in a browser. The `FRONTEND.md` file is the authoritative design spec (colour tokens, layout, component behaviour).

When the FastAPI server is running, `api/main.py` serves `index.html` at `GET /` and exposes the API at `/score`, `/labels`, and `/labels/import`. The frontend uses `fetch('/score', ...)` and falls back to the mock data automatically if the backend is unreachable (e.g. opened as a local file). API routes are registered before the static mount so they are never shadowed.

### Key dependencies

- `python-bitcoinlib` — raw tx parsing
- `bitcointx` — PSBT parsing (to be wired into `parser.py`)
- `requests` — mempool.space lookups in `lookup.py`
- `rich` — coloured CLI output
- `fastapi` + `uvicorn` — web API

---
> Source: [siri-score/siriscore](https://github.com/siri-score/siriscore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
