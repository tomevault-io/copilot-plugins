## nuxt-opentelemetry

> OpenTelemetry integration for Nuxt and Nitro.

# @scayle/nuxt-opentelemetry

OpenTelemetry integration for Nuxt and Nitro.

## Critical Constraints

- Uses `import-in-the-middle` for Node.js module hook registration. This ONLY works server-side. Browser builds MUST tree-shake this code.
- Instrumentation registers via `module.register()` at Nitro initialization, BEFORE application code runs.
- The `./instrumentation` export is for server entry points only. NEVER import it in client code.

---
> Source: [scayle/nuxt-opentelemetry](https://github.com/scayle/nuxt-opentelemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
