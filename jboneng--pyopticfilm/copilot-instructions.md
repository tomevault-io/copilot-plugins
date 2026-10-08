## model-lock

> Frozen model-lock tests must not be retargeted to fit other-model code


# Model lock tests

`tests/model_lock/` holds frozen **driver-path** oracles per hardware-validated model:

- `tests/model_lock/opticfilm_8200i_se/`
- `tests/model_lock/opticfilm_8100_v2/`

## Never without an explicit user request

Do **not** edit, delete, skip, xfail, loosen, parametrize, or move files under `tests/model_lock/` unless the user **explicitly** asks to update **that model's** lock tests.

Do **not** copy lock tests into `tests/` with weaker asserts to get a green run.

Do **not** add a lock folder for another model unless the user asks.

## When lock tests fail

- Change is for a **different** model: specialize that model (flags, leaf class, session branch). Leave other lock oracles unchanged.
- User is working on the **lock model's** driver: **stop and ask** before changing expected values. A failure may be a real regression.

## Allowed without asking

Read lock tests. Run `uv run pytest -m model_lock`. Change production code so lock tests keep passing.

---
> Source: [jboneng/pyopticfilm](https://github.com/jboneng/pyopticfilm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
