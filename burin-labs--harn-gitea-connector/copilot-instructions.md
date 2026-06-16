## harn-gitea-connector

> Use `harn-gitea-connector` when wiring Harn triggers or outbound helpers for Gitea.

# SKILL: harn-gitea-connector

Use `harn-gitea-connector` when wiring Harn triggers or outbound helpers for Gitea.

## What you get

- Provider id: `gitea`
- Trigger kinds: `webhook`
- Supported events: `push, pull_request, issues, issue_comment, release, repository, star`
- Webhook verification: `gitea_hmac`
- Outbound helpers: `api.request`, `pull_requests.comment`, `pull_requests.update`, `issues.comment`, `commit_status.set`, `repository_file.get`

## Trigger recipe

```toml
[[triggers]]
id = "gitea-events"
kind = "webhook"
provider = "gitea"
match = { path = "/hooks/gitea", events = ["pull_request.opened"] }
handler = "handlers::on_gitea_event"
secrets = { signing_secret = "gitea/signing-secret" }
```

For SourceHut, use a configured `public_key` secret instead of `signing_secret`.
For SVN polling, add a `poll` trigger and run `harn connector check . --provider svn --run-poll-tick`.

---
> Source: [burin-labs/harn-gitea-connector](https://github.com/burin-labs/harn-gitea-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
