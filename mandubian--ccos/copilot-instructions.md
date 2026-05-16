## no-specific-code-in-generic-code

> - Do not add fixes for custom demos/examples directly in generic modules (`arbiter`, `intent`, `runtime`, etc.).


# Keep Generic Code Example-Agnostic

- Do not add fixes for custom demos/examples directly in generic modules (`arbiter`, `intent`, `runtime`, etc.).
- Avoid hard-coded values, fixtures, or tests derived from a single example inside shared libraries; keep them alongside the example instead.
- If an example requires custom handling, isolate it in the example/capability layer and leave the generic code untouched.

---
> Source: [mandubian/ccos](https://github.com/mandubian/ccos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
