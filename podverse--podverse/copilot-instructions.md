## extensions-env

> Three-pillar env — Observability, Integrations, Extensions — and config mapping. Extensions env is last on main app containers.


# Platform env — extensions, integrations, observability

Use **extensions-env**, **integrations-web**, and **observability** skills for full detail.

## Three pillars

| Pillar | Config | Subsection order |
| ------ | ------ | ---------------- |
| Observability | `config.observability.*` | **First** |
| Integration | `config.integrations.<vendor>.<product>` | Second |
| Extension | `config.extensions.*` | **Last** (main app only) |

## Do

- Map extension toggles under `config.extensions.<service>.<property>` (e.g. `config.extensions.prometheus.enabled`).
- Use **`PROMETHEUS_*`** and **`OTEL_*`** for extensions/observability (no `EXT_*` prefix after plan 02).
- Mount **`podverse-integrations-config`** on runtime-config sidecars only.
- Mount **`podverse-extensions-config`** on main app + extension sidecar containers.
- Validate **Observability** categories before **Extensions** in startup validation.
- Express: observability HTTP middleware **before** extension metrics middleware.

## Don't

- Use `config.extensions.tracing` or extension toggles for always-on trace context.
- Put Integrations or Extensions env on runtime-config sidecars (Integrations CM + `NEXT_PUBLIC_*` only).
- Register `/extensions/prometheus/metrics` on Express/Next app processes (sidecar port **9464** only).
- Flatten integration config (`config.integrations.cloudflareWebAnalytics`).

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
