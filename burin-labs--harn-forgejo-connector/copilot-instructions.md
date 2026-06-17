## harn-forgejo-connector

> Use `harn-forgejo-connector` when wiring Harn triggers or outbound helpers for Forgejo and

# SKILL: harn-forgejo-connector

Use `harn-forgejo-connector` when wiring Harn triggers or outbound helpers for Forgejo and
Codeberg-style deployments.

## What you get

- Provider id: `forgejo`
- Trigger kinds: `webhook`
- Capabilities: `webhook`, `rate_limit`, `pagination`
- Supported events: `push, pull_request, issues, issue_comment, release, repository, star`
- Webhook verification: `gitea_hmac` (HMAC-SHA256 over the raw body via
  `std/connectors/shared::verify_hmac_signature`)
- Outbound helpers: `api.request`, `pull_requests.list`, `pull_requests.comment`,
  `pull_requests.update`, `issues.list`, `issues.comment`, `commit_status.set`,
  `repository_file.get`

## Trigger recipe

```toml
[[triggers]]
id = "forgejo-events"
kind = "webhook"
provider = "forgejo"
match = { path = "/hooks/forgejo", events = ["pull_request.opened"] }
handler = "handlers::on_forgejo_event"
secrets = { signing_secret = "forgejo/signing-secret" }
```

## Paginated list

```harn
let result = call(
  "pull_requests.list",
  {api_base_url: "https://codeberg.org/api/v1", access_token: token, owner: "acme", repo: "widgets", limit: 50},
)
// result.items, result.pages, result.complete, result.next_cursor
```

Pass `rate_limit = { capacity, refill_tokens, refill_interval_ms }` to override the preemptive
60 req/min token bucket, or `rate_limit = { disabled = true }` to bypass it.

---
> Source: [burin-labs/harn-forgejo-connector](https://github.com/burin-labs/harn-forgejo-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
