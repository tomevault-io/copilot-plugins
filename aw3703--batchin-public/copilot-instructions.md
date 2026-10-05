## batchin-public

> BatchIn is a verification-first developer platform. This repository contains public clients; the hosted control plane decides which models, routes, prices, and payment paths are currently enabled.

# BatchIn Agent Integration Manual

BatchIn is a verification-first developer platform. This repository contains public clients; the hosted control plane decides which models, routes, prices, and payment paths are currently enabled.

---

## Verified Public Discovery Resources

Before requesting account-gated services, autonomous agents should inspect the following reviewed endpoints:

- **Agent Guidance**: https://batchin.tech/agents.md
- **LLM Context**: https://batchin.tech/llms.txt
- **OpenAPI 3.1 Spec**: https://api.batchin.tech/openapi.json
- **Model Context Protocol (MCP) Manifest**: https://batchin.tech/.well-known/mcp
- **Server Card**: https://batchin.tech/.well-known/mcp/server-card.json
- **SDK Packages Registry**: https://batchin.tech/.well-known/sdk-packages.json
- **Public Model Catalog**: https://api.batchin.tech/v1/models

---

## Model availability

Do not embed a model matrix in an agent prompt or client release. Read the authenticated catalog at `https://api.batchin.tech/v1/models` and use only models whose response reports verified provider smoke, pricing, usage, and billing evidence for the current account. A model name, SDK fixture, or HTTP 200 response is not availability evidence.

---

## Agent Operational Constraints

1. **Authentication**: All customer API operations require `Authorization: Bearer <BATCHIN_API_KEY>`.
2. **Dedicated Capacity**: All compute resources are allocated as `Dedicated Capacity` or `Reserved Throughput` (TPS).
3. **No Synthetic Ledger Records**: Never generate synthetic VaaS receipts or fabricated cryptographic signatures as valid data.
4. **VaaS Verification**: Use `@batchin/vaas` or the `vaas_verify_receipt` MCP tool to confirm inference authenticity.
5. **Payment and settlement**: x402, USDC, Stripe, and chain anchoring are readiness-gated. Check the API readiness and ledger records before describing them as enabled or settled.

---
> Source: [aw3703/batchin-public](https://github.com/aw3703/batchin-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
