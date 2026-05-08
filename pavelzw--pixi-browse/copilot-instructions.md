## pixi-browse

> Lockfiles must be consistent with package metadata. After any change to `pixi.toml`, run `pixi lock`.

Lockfiles must be consistent with package metadata. After any change to `pixi.toml`, run `pixi lock`.

Everything runs in a pixi environment. Any command (like `pytest`) must be prefixed with `pixi run` (e.g. `pixi run pytest`).

Code formatting must align with our standards. Run `pixi run lint` before `git commit`s to ensure this.

When using things from py-rattler and pulling data, always properly type them, try to avoid using `Any`.

---
> Source: [pavelzw/pixi-browse](https://github.com/pavelzw/pixi-browse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
