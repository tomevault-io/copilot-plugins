## symperturb

> - Preserve the distinction between model-based virtual perturbation and causal intervention effects.

# Repository guidance for AI coding agents

- Preserve the distinction between model-based virtual perturbation and causal intervention effects.
- Treat `.agents/skills/symperturb-analysis/references/method-specification.md` as the method contract.
- Do not add robustness to confirmatory VPPS.
- Keep exact-vKO handling explicit; never standardize a constant target column.
- Preserve signed combination increments by default.
- Add tests for any change to state moments, utilities, normalization, topology propagation, or bootstrap ranking.
- Run `pytest` and the example CLI before proposing a release.

---
> Source: [zfrory15-max/SymPerturb](https://github.com/zfrory15-max/SymPerturb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
