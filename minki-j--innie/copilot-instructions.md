## innie

> **Builder pattern:** Config objects are `chz` dataclasses (SupervisedDatasetBuilder, RLDatasetBuilder, EnvGroupBuilder). They expose `.build()`/`__call__()` returning runtime objects.

## Architecture

**Builder pattern:** Config objects are `chz` dataclasses (SupervisedDatasetBuilder, RLDatasetBuilder, EnvGroupBuilder). They expose `.build()`/`__call__()` returning runtime objects.

**Key code locations:**
- SL: `tinker_cookbook/supervised/train.py`
- RL: `tinker_cookbook/rl/train.py`
- DPO: `tinker_cookbook/preference/train_dpo.py`
- Renderers: `tinker_cookbook/renderers/`
- Completers: `tinker_cookbook/completers.py`
- RL types: `tinker_cookbook/rl/types.py`
- Recipes: `tinker_cookbook/recipes/`

---

## Conventions

**Subscript suffixes** for tensor names: `_P` (problems), `_G` (groups), `_T` (tokens), `_D` (datums). Example: `tokens_P_G_T[p][g][t]`

**Code style:**
- Explicit typing; avoid `Any` / `type: ignore`
- Use `safezip`, `timed`, `scope` helpers
- `@chz.chz` decorator for config serialization
- `ml_log.log_metrics` for metrics; `logtree` for transcripts

**Env lifecycle:** `Env` objects are single-use (no reset). Create via `EnvGroupBuilder`.

---

## Common Pitfalls

1. **LoRA LR:** Use `hyperparam_utils.get_lr(model_name)` - LoRA needs ~10x higher LR than full fine-tuning.

2. **Renderer mismatch:** Match `renderer_name` to model family (`llama3`, `qwen3`, `role_colon`).

3. **Async gaps:** Submit `forward_backward_async` and `optim_step_async` back-to-back before awaiting.

4. **Sampler desync:** Create a **new** sampling client after saving weights.

5. **Type construction:** Use helper functions, not manual dict construction. See `supervised/data.py` and `supervised/common.py`.

6. **Group semantics:** RL advantages are centered within each group.

7. **DPO:** Start with `dpo_beta=0.1`, LR~1e-5.

---

## Testing

```bash
# Unit tests (no API needed)
pytest tinker_cookbook/tests/test_renderers.py
pytest tinker_cookbook/tests/test_utils.py

# Smoke tests (requires API key + network)
pytest tinker_cookbook/tests/smoke_tests.py
```

For debugging, shrink workloads via `n_batches`, `batch_size`, `group_size` in dataset builders.---
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/minki-j) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
