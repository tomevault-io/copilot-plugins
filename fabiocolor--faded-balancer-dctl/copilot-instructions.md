## dctl-style

> - **Precision**: Prefer `float` operations; avoid implicit type narrowing.

### DCTL Coding Guide

- **Precision**: Prefer `float` operations; avoid implicit type narrowing.
- **Branching**: Minimize dynamic branching inside pixel loops; use lerp/mix where possible.
- **Parameters**: Keep names/defaults aligned with [presets/presets.json](mdc:presets/presets.json) and [README.md](mdc:README.md).
- **Helpers**: Reuse utilities from [internal/docs/DCTL_HELPERS.md](mdc:internal/docs/DCTL_HELPERS.md) and vendor [ColorConversion.h](mdc:internal/docs/vendor/DaVinciCTL/ColorConversion.h) when applicable.
- **Performance**: Avoid repeated expensive math; precompute constants; clamp inputs as needed.
- **Safety**: Handle out-of-gamut and NaNs defensively; clamp to [0, 1] where appropriate.
- **Clarity**: Use descriptive variable names; early returns for error paths; keep nesting shallow.
- **Docs**: Update [docs/FAQ.md](mdc:docs/FAQ.md) if behavior changes that affect users.

---
> Source: [fabiocolor/Faded-Balancer-DCTL](https://github.com/fabiocolor/Faded-Balancer-DCTL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
