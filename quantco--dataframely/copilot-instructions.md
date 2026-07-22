## dataframely

> Lockfiles must be consistent with package metadata. After any change to `pixi.toml`, run `pixi lock`.

Lockfiles must be consistent with package metadata. After any change to `pixi.toml`, run `pixi lock`.

Everything runs in a pixi environment. Any command (like `pytest`) must be prefixed with `pixi run` (e.g. `pixi run pytest`).

Code formatting must align with our standards. Run `pixi run lint` before `git commit`s to ensure this.

---
> Source: [Quantco/dataframely](https://github.com/Quantco/dataframely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
