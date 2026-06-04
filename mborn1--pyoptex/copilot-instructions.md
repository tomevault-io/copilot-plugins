## environment

> Known environment and dependency caveats


# Environment caveats

- **kaleido** (plotly static export): can emit harmless dbus errors in headless/Cloud VM; safe to ignore.
- **numba** (~0.61): pulls in llvmlite; first import may have a short JIT compilation delay.

---
> Source: [mborn1/pyoptex](https://github.com/mborn1/pyoptex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
