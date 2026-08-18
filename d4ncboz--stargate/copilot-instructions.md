## stargate

> 1. Install dependencies only when `package.json` changes: `npm install`.

# AGENTS.md

## Setup checklist

1. Install dependencies only when `package.json` changes: `npm install`.
2. Run the baseline: `npm test`.
3. Validate Worker syntax: `node --check src/index.js`.
4. Use `npx wrangler dev` for local Worker testing.
5. Never deploy until tests and secret scan pass.

## Canonical workflows

- Change validation or access logic: edit `src/core.js`, add a test in `test/core.test.js`, run `npm test`.
- Change routes or GitHub API behavior: edit `src/index.js`, run tests and `node --check src/index.js`.
- Change persistence: update `schema.sql`; use an additive migration for an existing production database.
- Deploy: run `npx wrangler deploy`, then verify the public endpoint with `curl`.

## Exit codes

| Code | Meaning |
|---:|---|
| 0 | command completed successfully |
| 1 | test, syntax, deployment, or API operation failed |

## Environment and bindings

| Name | Kind | Purpose |
|---|---|---|
| `GITHUB_CLIENT_ID` | variable | public OAuth application identifier |
| `GITHUB_CLIENT_SECRET` | secret | OAuth code exchange |
| `SESSION_SECRET` | secret | HMAC signing for OAuth state |
| `STATE` | KV | short-lived server-side OAuth sessions |
| `DB` | D1 | gate records |

## Hard constraints

- Never commit secrets, tokens, cookies, account exports, or `.dev.vars`.
- Never collect GitHub passwords, OTPs, passkeys, or recovery codes.
- Never perform follow or star without an explicit labeled user action.
- Never claim that a gate makes a public repository private.
- Preserve server-side verification before redirecting.
- Keep creator repository ownership validation.

## Error handling

- GitHub `401`: discard the stale session and restart OAuth.
- GitHub `403`: show authorization or rate-limit failure; do not treat it as success.
- GitHub `404`: keep the target hidden; treat the requirement as unmet or repository unavailable.
- D1 failure: return an error; never manufacture a gate ID.
- Missing secret or binding: stop deployment and fix configuration.

---
> Source: [d4ncboz/stargate](https://github.com/d4ncboz/stargate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
