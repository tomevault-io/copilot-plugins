## hippohubapiservice

> Ensure new API endpoints include importable Postman requests


# Postman Collection for New Endpoints

- Every newly added API endpoint must include a Postman-importable request in the same delivery.
- Do not mark endpoint work complete if Postman import data is missing.

## Required Output

- Maintain collection file at `docs/postman/hippohub.postman_collection.json`.
- If the collection file does not exist yet, create it with valid Postman v2.1 schema.
- Add one request item per new endpoint, grouped by module (`User`, `Wallet`, `Game`).
- Use `{{HIPPOHUB}}` as the Postman domain variable for host/prefix, never hardcode production secrets.

## Request Content Rules

- Include method, full path, headers, and sample body aligned with API contract.
- Add auth-related headers for protected APIs (for example token/signature/timestamp/nonce placeholders).
- Add at least one success example payload and one failure example payload when feasible.
- Request names must use Chinese and remain stable: `<模块中文> - <动作中文>`.
- Prefer Swagger Chinese summary as request action name; if summary is not Chinese, provide explicit Chinese naming in the collection update.

## Verification Gate

- Before finishing endpoint implementation, verify the new endpoint is present in the Postman collection JSON.
- Validate JSON is syntactically correct and importable by Postman.
- Ensure request path and payload stay consistent with Swagger docs and route registration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lhzptg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
