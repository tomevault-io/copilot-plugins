## 060-security

> Secrets, Cognito env, and fail-closed auth.


# Security

Read `docs/SECURITY.md`.

- No hardcoded secrets or JWTs in source.
- Cognito IDs via env / CI / DefinePlugin only.
- Do not log full tokens.
- Auth failures must deny (throw `Unauthorized`).

---
> Source: [jessemull/100-letters-project-authorizer](https://github.com/jessemull/100-letters-project-authorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
