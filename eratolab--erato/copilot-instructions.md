## secure-api-routes

> Describes how to secure API routes

- API routes should run appropriate `authorize` checks with the `PolicyEngine`
- The policy structure and actions are described via Open Policy Agent Rego in [backend.rego](mdc:backend/policy/backend/backend.rego)
- If available resource kinds or actions need to be extended, this can be done in [types.rs](mdc:backend/src/policy/types.rs)

---
> Source: [EratoLab/erato](https://github.com/EratoLab/erato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
