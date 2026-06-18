## linear-api

> Talk to Linear's GraphQL API directly via a thin Python CLI, with progressive-discovery references for issues, projects, comments, webhooks, cycles, workflow states, and admin. Use this instead of the Linear MCP when you want full coverage (webhooks, cycle/state mutations, audit log, admin) or want to avoid loading ~40 tool schemas into context.


# linear-api

You are talking to **Linear** (linear.app — the issue tracker). This skill is the alternative to the Linear MCP: instead of pre-loaded tool definitions, you compose GraphQL operations and run them through `scripts/linear.py`. You only learn the bits of Linear you need for the task in front of you.

## When to use

Use this skill when the user asks to:

- read or change Linear data (issues, projects, comments, documents, cycles, etc.)
- set up or manage **webhooks** (the MCP can't)
- manage **cycles** or **workflow states** as writes (the MCP is read-only on both)
- create / update **issue relations**, reactions, notification subscriptions, templates, custom views, favorites
- query the **audit log** or call admin mutations (user roles, org invites)

If the user already has the Linear MCP loaded and the task is plain read or simple writes (`save_issue`, `list_projects`), the MCP is fine — this skill earns its place on the gap surface and on token-cost-sensitive workflows.

## First-run checklist

1. Confirm `LINEAR_API_KEY` is set: `echo "${LINEAR_API_KEY:0:8}"` should show `lin_api_`. If not, see `references/auth.md`.
2. From this directory: `uv sync` (one-off, installs httpx + python-dotenv).
3. Smoke check: `uv run python scripts/linear.py query 'query { viewer { id name email } }'`.
4. You're live. Pick the reference file matching the task and load only that one.

## Default team

If the user does not name a team, default to **AgileAndy** (key `AGI`). Resolve its UUID once per session and reuse:

```graphql
query { teams(filter: { name: { eq: "AgileAndy" } }) { nodes { id key name } } }
```

Cache the returned `id` for the rest of the session. Only override when the user explicitly names a different team.

## Reference file index — load only what you need

| File | Load when the task is about… |
|---|---|
| `references/auth.md` | Getting a key, switching to OAuth, token revocation, scopes |
| `references/schema-summary.md` | Entity model, identifiers, pagination, filtering, save / delete semantics |
| `references/rate-limits.md` | 429s, complexity errors, retry strategy, observed limits |
| `references/common-queries.md` | Day-to-day reads + writes — issues, projects, comments, docs (Phase 1 MCP-parity surface) |
| `references/webhooks.md` | Subscribing to issue / comment / project / cycle events |
| `references/mutations-cheatsheet.md` | The "gap five" — cycles, workflow states, relations, templates, admin |
| `references/cycles-daily-automation.md` | Running 1-cycle-per-day with rollover + 14-day rolling roadmap (uses `scripts/cycles_maintain.py`) |
| `references/prune-archive.md` | Querying the local sqlite archive of pruned issues (used by `/linear-prune` and `scripts/linear_prune.py`) |

Don't load every file. Load `schema-summary.md` for orientation and one task-specific file. If you're stuck on an unknown type, run `scripts/linear.py introspect <TypeName>` rather than loading more references.

## CLI

```bash
uv run python scripts/linear.py query     <doc-or-file> [--variables JSON|@file.json]
uv run python scripts/linear.py mutation  <doc-or-file> [--variables JSON|@file.json]
uv run python scripts/linear.py introspect <TypeName>

# Daily-cycle roadmap maintainer (idempotent; cron this once a day past AWST midnight).
uv run python scripts/cycles_maintain.py --team AGI [--days-ahead N] [--auto-assign] [--dry-run]
```

- `<doc-or-file>` is either an inline GraphQL string or a path to a `.graphql` file.
- Rate-limit headers (`x-ratelimit-*`) and `x-complexity` are logged to **stderr** after every call.
- 429 raises and exits non-zero with `retry-after` in the message. The CLI does not auto-retry.
- Personal keys (`lin_api_…`) go in `Authorization` raw; OAuth tokens (`lin_oauth_…`) get a `Bearer ` prefix automatically.

## One worked example end-to-end

Task: "Create a P1 bug in the AGI team titled `auth: 401 on token refresh`, then list its assignee and state."

```bash
# 1. Find the team's UUID (need it for the mutation).
uv run python scripts/linear.py query \
  'query { teams(filter: { key: { eq: "AGI" } }) { nodes { id key } } }'
# → { "teams": { "nodes": [ { "id": "a57f5901-…", "key": "AGI" } ] } }

# 2. Create the issue. Priority 1 = Urgent.
uv run python scripts/linear.py mutation \
  'mutation Create($input: IssueCreateInput!) {
     issueCreate(input: $input) {
       success
       issue { id identifier title priority url }
     }
   }' \
  --variables '{"input": {
     "teamId": "a57f5901-…",
     "title": "auth: 401 on token refresh",
     "priority": 1
   }}'
# → { "issueCreate": { "success": true, "issue": { "identifier": "AGI-87", … } } }

# 3. Read it back with assignee + state.
uv run python scripts/linear.py query \
  'query Get($id: String!) {
     issue(id: $id) {
       identifier title priority
       assignee { name email }
       state { name type }
     }
   }' \
  --variables '{"id": "<id-from-step-2>"}'
```

That's the pattern: introspect or query for an id you don't have, run the mutation with variables, read it back. Three calls, ~8 complexity points total.

## Patterns to follow

- **Always select `success` on mutations.** Linear can return `success: false` with no error array. Check it.
- **Pass IDs as variables, not string interpolation.** GraphQL variables are typed and avoid injection.
- **Page with `nodes` + `pageInfo.endCursor`.** Don't fetch `first: 250` if you only need 10.
- **For unknown types, introspect one type at a time.** `__schema` will trip the complexity cap.
- **Soft vs hard delete is a real distinction.** `issueDelete(id)` trashes; add `permanentlyDelete: true` to purge.

## What this skill does NOT do (yet)

- Auto-retry on 429 / complexity errors. Caller decides.
- Bulk operations beyond what GraphQL exposes natively (`issueBatchCreate` is reachable; client doesn't add a multi-call wrapper).
- OAuth flows. Personal API key only until Phase 3.
- Webhook receiver scaffolding. The skill manages webhook *subscriptions*; you bring your own HTTP endpoint.

## Where to look first when something breaks

| Symptom | Look here |
|---|---|
| `error: LINEAR_API_KEY not set` | `references/auth.md` |
| `HTTP 429` or `rate limited` | `references/rate-limits.md` |
| `field "x" not allowed on type "Y"` | Run `linear.py introspect Y`; check `references/schema-summary.md` |
| `Entity not found` after a delete | Default delete is soft — see `schema-summary.md`'s soft-vs-hard section |
| Mutation returns `success: false` with empty error array | The op was rejected silently; introspect the input type for required fields you missed |

---
> Source: [agileandy/linear-api](https://github.com/agileandy/linear-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
