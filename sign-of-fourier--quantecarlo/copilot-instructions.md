## quantecarlo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Cleanup plan

See `TECH_DEBT.md` in this repo for the staged cleanup plan, repo map, and
critical conventions. Read that before starting any new session on this codebase.

## Setup and Development

```bash
pip install -e .          # install in editable mode (hard deps: optuna>=3.0, numpy, scipy)
pip install optunahub     # needed to run demos
python demos/demo.py      # ask-tell demo using BatchSampler + modal_suggest (requires Modal endpoint)
```

Run tests:
```bash
python -m pytest tests/ -v -k "not Live"          # unit tests only
python -m pytest tests/test_modal_live.py -v -s   # requires Modal endpoint
```

No linter configuration in this repo.

## Architecture

`quantecarlo` provides two `suggest_fn` implementations for the optunahub `BatchSampler`:

1. **`fantasize_suggest`** (`quantecarlo/_fantasize.py`) — self-contained in-process GP.
   Fits an RBF GP using numpy/scipy, selects a batch by Expected Improvement with
   fantasization. No remote service required. This is the primary example in the
   optunahub `batch_sampler` package.

2. **`modal_suggest`** (`quantecarlo/bo_sampler.py`) — delegates to the remote Modal GP
   service (`~/projects/boaz/modal/modal_gp_api.py`). Generates `n_candidates` random
   points client-side, POSTs them with observations to the endpoint, returns the
   highest q-EI batch. Use via `functools.partial` to bind `api_url` and other params.

Both follow the `suggest_fn` contract of the optunahub `BatchSampler`:

```
suggest_fn(X: list[list[float]], y: list[float], search_space: list[DimSpec], q: int)
    -> list[dict[str, Any]]   # exactly q parameter dicts
```

`BatchSampler` owns the lock, cache, startup fallback, and threading. The `suggest_fn`
just does the math / HTTP call.

3. **`call_modal_api` / `call_modal_api_multioutput`** (`quantecarlo/_modal_api.py`) —
   the raw Modal HTTP client. Takes numpy arrays directly (higher = better, no direction
   param). Used internally by `modal_suggest` and re-exported by `meta-ads-demo`. This
   is the **single source of truth** for the Modal API contract — update here when the
   API payload changes.

### `DimSpec` (`quantecarlo/bo_sampler.py`)

`@dataclass` describing one search-space dimension. Fields: `name`, `type` (`"float"` |
`"int"`), `low`, `high`, `log`, `step`. Shared between both suggest functions and the
optunahub `BatchSampler`.

### `modal_suggest` (`quantecarlo/bo_sampler.py`)

```python
modal_suggest(X, y, search_space, q, *, direction="minimize", api_url, n_candidates=512,
              train_steps=60, lr=0.1, xi=0.01, mode="production", seed=None, timeout=120.0)
```

**y convention**: `BatchSampler` passes raw study values in the study's direction
convention (lower = better for minimize, higher = better for maximize). Pass
`direction` matching your Optuna study direction — `modal_suggest` handles conversion
to the Modal API's higher-is-better convention internally. Bind via `functools.partial`.

**Payload sent**: `{X, y_higher_is_better, candidates, q, n_batches, train_steps, lr, xi, mode}`

**Response parsed**: `{"candidates": [{"index": int, "x": [...], "mu": float, "sigma": float}, ...]}`

### `fantasize_suggest` (`quantecarlo/_fantasize.py`)

```
fantasize_suggest(X, y, search_space, q, direction="minimize", n_candidates=512, noise=1e-3, xi=0.01, seed=None)
```

Takes `y` in the study's direction convention (same as `modal_suggest`). Negates internally
for minimize studies so the GP always works higher-is-better. EI formula is maximize convention throughout.

### `_modal_api.py` (`quantecarlo/_modal_api.py`)

```python
call_modal_api(api_url, X, y, candidates, q, n_batches, train_steps, lr, xi, mode, timeout)
call_modal_api_multioutput(api_url, X, y, candidates, d_train, d_cands, rho, q, ...)
```

Both take numpy arrays, return `list[dict]` with `index`, `x` (np.ndarray), `mu`, `sigma`.
`call_modal_api_multioutput` adds `d_train`, `d_cands`, `rho` for the cross-platform GP path.

### Related repos

| Repo | Role |
|---|---|
| `~/projects/boaz/modal/modal_gp_api.py` | The Modal GP server — `modal_suggest` POSTs here |
| `~/projects/optunahub-registry/package/samplers/batch_sampler/` | `BatchSampler` — wraps any `suggest_fn` with lock + cache |
| `~/projects/meta-ads-demo/backend/bo_pipeline/modal_bo.py` | Re-exports `call_modal_api` / `call_modal_api_multioutput` from quantecarlo; owns PCA helpers and env-var config |

### demos/

- **`demo.py`** — NAS on the breast-cancer dataset using an MLP. Uses `BatchSampler` +
  `modal_suggest` with an explicit ask-tell loop so the batching contract is visible.
  Requires a running Modal endpoint and `optunahub`.
- **`demo7.py`** — Compares BatchSampler (ask-tell) against Optuna's default TPE
  (`n_jobs=4`) on a pool-based image ad search task. Requires external data files not
  included in this repo.

### API contract (Modal endpoint)

Endpoint: `POST https://markshipman4273--bo-gp-service-gp-suggest.modal.run`

Request fields: `X`, `y` (higher = better), `candidates` (actual pool vectors),
`q`, `n_batches`, `train_steps`, `lr`, `xi`, `mode`.

Response: `{"candidates": [{"index": int, "x": [...], "mu": float, "sigma": float}]}`

Full spec: `~/projects/boaz/modal/API.md`

---
> Source: [sign-of-fourier/quantecarlo](https://github.com/sign-of-fourier/quantecarlo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
