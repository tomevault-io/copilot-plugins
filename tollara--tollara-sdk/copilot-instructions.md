## project-overview

> - Product: Tollara — SaaS for marketing/monetizing/managing AI agents and MCP servers; SDKs support callers (invoke, validate keys) and backends (verify HMAC, user context, usage/progress/completion).


- Product: Tollara — SaaS for marketing/monetizing/managing AI agents and MCP servers; SDKs support callers (invoke, validate keys) and backends (verify HMAC, user context, usage/progress/completion).
- Services the SDKs touch (only three): Gateway (invoke), Core (service key validation), Usage (report/progress/complete, quota). Base URLs are configurable; ECS uses /{service}/api/v1-style prefixes; local defaults differ (see doc table).
- Not in scope for these SDKs: Security/Cognito for SDK flows, Messaging, Gateway-usage-consumer — callers use service keys (Bearer) for gateway invoke and validate.
API surface: Follow the doc’s table (invoke sync/async, validate, report, progress, completion); append paths to the correct service base; async returns requestId, progressUrl, callbackUrl.
- Gateway → backend headers: X-Tollara-Signature, X-Tollara-Timestamp, X-Tollara-User-ID, X-Tollara-Plan, X-Tollara-Roles, X-Tollara-Subscription-Active.
- HMAC: HMAC-SHA256, UTF-8, Base64; inbound canonical = payload + timestamp + userContextString (userContext as specified); outbound (usage/progress/complete) = bodyString + timestamp; validate response = HMAC(responseBody + timestamp, serviceSecret); use constant-time compare; optional timestamp skew window.
Reference implementation: Java in this repo (sdk-java + vendored lib) is the source of truth for contracts and HMAC — mirror in other languages (TollaraRequestVerifier, UsageServiceClient, ServiceKeyValidationClient, HmacUtils, models).
Naming: Display Tollara; packages/artifacts use tollara (lowercase) per ecosystem conventions in the doc.
Repo layout: docs/ (specs, vectors); sdk-* per language; integration-n8n, integration-openclaw; each folder self-contained (no shared polyglot runtime).
Plan / checklist: Specs + test vectors in docs; unified surface + README per SDK/integration; CI per folder; no leftover platform-only deps in Java; publish placeholders as in plan.

---
> Source: [tollara/tollara-sdk](https://github.com/tollara/tollara-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
