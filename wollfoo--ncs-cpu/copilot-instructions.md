## ncs-cpu

> activation: always_on

---
trigger: always_on
---

---
type: guardrail
scope: project
priority: high
activation: always_on
---

# COMPLIANCE & PII PROTECTION
- Mask email, phone, ID numbers in outputs
- Never log real secrets; use `<REDACTED>`
- GDPR/CCPA reminder for user data handling
- Suggest vault/secret manager for credentials

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wollfoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
