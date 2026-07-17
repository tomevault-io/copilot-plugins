## scalability

> Scalability and performance practices for sensor/CSV data processing


# Scalability

Write code that scales with large FSR/sensor CSV logs and many modules.

- Avoid loading entire large datasets into memory when avoidable. Prefer streamed/chunked reads and vectorized pandas/numpy operations over Python row loops for sensor data.
- Cache expensive computations with Streamlit caching (`@st.cache_data` for data/DataFrames, `@st.cache_resource` for models/connections) so reruns don't recompute.
- Do not duplicate recomputation across modules. Compute shared analysis once, publish via `ctx.set_param(...)`, and have other modules read it with `ctx.param(...)` instead of recomputing.
- Keep per-module `render(ctx)` efficient: gate heavy work behind user actions/flags, and return early when a module is disabled or has no data.
- Design module interfaces so new analyses can be added without modifying `core/`. Extend behavior by adding modules + `ENABLED_MODULES` entries, not by editing the loader/registry.
- Push domain logic into `core/domain/` / `myprosole_analysis/` as pure functions that operate on arrays/DataFrames, keeping them fast and independently testable.

---
> Source: [MyProEye-UG/MyProSole](https://github.com/MyProEye-UG/MyProSole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
