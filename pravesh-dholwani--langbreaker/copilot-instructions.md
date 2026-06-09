## langbreaker

> Guidance for Claude Code (and humans) working in this repo.

# CLAUDE.md

Guidance for Claude Code (and humans) working in this repo.

## What this project is

`lc-circuit-breaker` adds **statefulness** to LangChain's `with_fallbacks`. Vanilla LangChain retries the primary model on every request — if the primary is dead and times out at 3s, every call pays 3s before falling through. This library wraps the chain in a distributed circuit breaker so OPEN models are skipped immediately.

## Drop-in API

The only line a consumer changes:

```python
# before
model = primary.with_fallbacks([fallback])

# after
from lc_circuit_breaker import with_stateful_fallbacks
model = with_stateful_fallbacks(primary, [fallback])
```

## Project layout

```
src/lc_circuit_breaker/
  __init__.py         # public re-exports
  state.py            # State enum, FailureKind, Snapshot, Decision
  config.py           # CircuitConfig dataclass + env loading (LCCB_*)
  errors.py           # AllCircuitsOpen, StoreUnavailable
  classifier.py       # default exception -> FailureKind
  identity.py         # default circuit_id derivation
  breaker.py          # CircuitBreaker (per-identity coordinator)
  registry.py         # CircuitRegistry + pub/sub bridge task
  runnable.py         # StatefulFallbackRunnable (LangChain Runnable)
  factory.py          # with_stateful_fallbacks() entry point
  cli.py              # `lccb` ops CLI (click)
  store/
    base.py           # StateStore Protocol + StoreDecision
    memory.py         # InMemoryStateStore (single process / tests / Redis-down fallback)
    redis.py          # RedisStateStore (production)
    lua/              # atomic Lua scripts — single source of truth for state transitions
      decide.lua
      record_failure.lua
      record_success.lua
      force_state.lua
tests/
  conftest.py
  test_breaker.py     # state machine, probe lock, window eviction
  test_classifier.py  # exception mapping
  test_runnable.py    # LangChain integration
```

## Key invariants — do not break these

1. **Redis is the source of truth.** Local snapshots are a 250ms-TTL optimization. Pub/sub is invalidation, not state. If you find yourself trusting a local snapshot for a transition decision, you're doing it wrong.
2. **All state transitions are atomic Lua scripts.** Never do read-modify-write in Python against Redis. New transitions belong in `store/lua/`.
3. **The probe lock is what prevents thundering herd.** Only one pod across the cluster issues the HALF_OPEN probe per cooldown cycle. If you change probe semantics, re-read §8 of `PLAN.html`.
4. **Time is read from the store, not from each pod's clock.** Lua uses `ARGV[now_ms]` passed by Python, but cooldown comparisons happen *inside* Lua so they're consistent. Don't compare local `time.time()` against `opened_at_ms` outside Lua.
5. **Hash-tagged keys.** All per-identity Redis keys use `cb:{ns}:{identity}:...` (the `{identity}` is a Redis Cluster hash tag). All keys touched by one Lua script must share the same tag.
6. **`PASSTHROUGH_KINDS` never trip the breaker and never fall through.** BAD_REQUEST, CONTENT_FILTERED, CANCELLED bubble straight to the caller. The classifier is the only place this distinction is made.

## Configuration

All knobs live in `CircuitConfig` (see [config.py](src/lc_circuit_breaker/config.py)). Every field has a matching `LCCB_*` env var. Explicit kwargs > env > defaults. Validation runs in `__post_init__`.

Defaults: 5 failures in 30s trip → 20s cooldown → 1 probe → close. Exponential backoff capped at `max_cooldown_seconds`.

## Confirmed product decisions

These were decided up-front; revisit only with the user:

- **429s count toward the breaker** (RATE_LIMITED is in `COUNTING_KINDS`).
- **The probe uses the real user request**, not a synthetic prompt.
- **All circuits open ⇒ raise `AllCircuitsOpen`**, not a sentinel return.
- **Ops surface is the `lccb` CLI** (`lccb status / open / close / reset / watch`). No HTTP admin endpoint.
- **No observability layer in v1.** Metrics/tracing are out of scope. Add `log.info/debug` only — don't introduce a metrics dependency.

## Adding a state transition

If you need a new transition (e.g. a fourth state, or a different close condition):

1. Edit the relevant Lua script in `store/lua/` — keep the operation atomic.
2. Mirror the logic in `store/memory.py` so single-process tests still cover it.
3. Update `State` / `Snapshot` if new fields are needed.
4. Add a test in `tests/test_breaker.py` that runs against `InMemoryStateStore`.
5. If you change the Lua return shape, update `RedisStateStore._parse_snapshot` / `_parse_decide`.

## Adding a new error type

1. Edit `classifier.py`. Match by exception class name first (it works across SDK versions and avoids importing optional deps), then status code as fallback.
2. Decide: counts toward breaker (add to `COUNTING_KINDS`) or passthrough (`PASSTHROUGH_KINDS`).
3. Add a case in `tests/test_classifier.py`.

## Running tests

```bash
python3 -m venv .venv
.venv/bin/pip install -e '.[dev]'
.venv/bin/pytest tests/
```

The default suite uses `InMemoryStateStore` only — no Redis required. A future integration suite against real Redis should go in `tests/integration/` with a `redis` marker.

## What's intentionally out of scope (v1)

- **Streaming / batch edge cases.** `stream`/`astream` aren't implemented on the wrapper. `batch`/`abatch` exist but just call invoke per-item (no probe coordination across the batch). If you need streaming, see §11.4 of PLAN.html for the design.
- **Observability** (Prometheus, OTEL spans, admin HTTP). The hooks exist (we log on transitions); adding metrics is a future PR.
- **Adaptive thresholds.** N and S are static config.
- **Per-tenant circuits.** Identity is per-model only. Override via `config.circuit_id_fn` if needed.

## Common gotchas

- `invoke()` from inside a running event loop raises. Tell the user to use `ainvoke()`. We don't ship `nest_asyncio` magic on purpose.
- `redis.asyncio` 5.x renamed `close` to `aclose` — `RedisStateStore.close()` tries both.
- The pub/sub channel is **per-identity** (`cb:{ns}:{id}:events`) and the subscriber uses PSUBSCRIBE on `cb:{ns}:*:events`. This is to keep PUBLISH inside Lua valid on Redis Cluster. Don't "simplify" to a single channel — Cluster will reject the cross-slot PUBLISH.
- `snapshot_ttl_ms=0` in tests disables the local cache so behavior is deterministic. Production default is 250ms; don't set it to 0 there.

## Useful files when debugging

- A breaker getting stuck OPEN forever → check `effective_cooldown` in the state hash; backoff cap might be too high.
- A model that should fall through but errors out → check `classifier.py` first; the exception probably maps to `BAD_REQUEST` (passthrough).
- Multiple pods all probing → probe lock isn't being acquired; check `cb:{ns}:{id}:probe_lock` TTL and holder.

---
> Source: [pravesh-dholwani/langbreaker](https://github.com/pravesh-dholwani/langbreaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
