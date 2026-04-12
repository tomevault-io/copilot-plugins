## normix

> normix project identity and distribution catalog


# normix

JAX package for Generalized Hyperbolic distributions as exponential families.
Built on Equinox. Dependencies: `jax`, `equinox`, `jaxopt`, `tfp` (Bessel only).

| Distribution | Subordinator | Stored Attributes |
|---|---|---|
| Gamma | — | `alpha`, `beta` |
| InverseGamma | — | `alpha`, `beta` |
| InverseGaussian | — | `mu`, `lam` |
| GIG | — | `p`, `a`, `b` |
| MultivariateNormal | — | `mu`, `L` (Cholesky) |
| VarianceGamma | Gamma | `mu`, `gamma`, `L`, `alpha`, `beta` |
| NormalInverseGamma | InverseGamma | `mu`, `gamma`, `L`, `alpha`, `beta` |
| NormalInverseGaussian | InverseGaussian | `mu`, `gamma`, `L`, `mu_ig`, `lam` |
| GeneralizedHyperbolic | GIG | `mu`, `gamma`, `L`, `p`, `a`, `b` |

All distributions provide: `pdf`, `cdf` (where analytical), `mean`, `var`, `std`, `rvs`.
Joint mixtures provide `rvs(n, seed) → (X, Y)`. Marginals provide `rvs`, `mean`, `cov`.

Bessel: `log_kv(v, z)` with `@jax.custom_jvp` in `_bessel.py`.
Architecture: `docs/ARCHITECTURE.md`. Design: `docs/design/jax_design.md`.
Tech notes: `docs/tech_notes/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xshi19) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
