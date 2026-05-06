## domain-knowledge

> Domain concepts — read this before auditing or reasoning about business entities


# Domain Knowledge

## billing_entity is a customer account, not SaaS billing

`BillingEntity` in this app is a **contractor/customer billing account** — the entity a job or withdrawal gets billed to. It has nothing to do with billing the app's operator (there is no in-app SaaS billing).

`billing_entity_id` on `ContractorContext`, `Withdrawal`, `Job`, `Payment`, etc. refers to this customer account FK.

`CurrentUser` does NOT have `billing_entity_id` — that field belongs to contractor user records in the `operations` context, not on the JWT-derived user identity. A contractor's `billing_entity_id` is resolved from their DB record, not from the token.

Do not conflate these concepts when auditing auth, CurrentUser, or financial flows.

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
