## share

> Share is an OSS, deploy-to-your-own-Cloudflare temporary file-transfer app. Public demo mode uses scoped capability links, not identity claims.

# Share agent guide

Share is an OSS, deploy-to-your-own-Cloudflare temporary file-transfer app. Public demo mode uses scoped capability links, not identity claims.

## Canonical commands

```sh
bun install
bun run typecheck
bun run test
bun run dry-run
bun run check
bun run browser:e2e # after starting local Worker with Turnstile test keys
```

## Public security rules

- Never expose an R2 public bucket URL.
- Never put capability tokens in request paths or query parameters; fragments stay browser-side and API calls use `Authorization: ShareCapability`.
- Never add unbounded retention, public listing, inline previewing, or unverifiable email ACLs to demo mode.
- Keep Turnstile, storage quotas, download byte budgets, and expiration enforced before a public deployment.
- Identity-backed confidential sharing is a separate deployment profile, not a silent fallback.

## Deployment boundary

The hosted demonstration target is eventually `share.coey.dev` in Jordan's personal Cloudflare account. Do not push or deploy without explicit approval.

---
> Source: [acoyfellow/share](https://github.com/acoyfellow/share) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
