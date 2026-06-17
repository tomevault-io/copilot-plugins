## jobber-cli

> |


# Jobber

Use this skill whenever you need to inspect or operate Jobber through the local `jobber` CLI.

Important naming detail:

- npm package name: `jobber-cli`
- CLI binary name: `jobber`

Resolution order:

1. If `jobber` is already on `$PATH`, use it directly.
2. Otherwise run the published package explicitly with `npx -y jobber-cli <args>`.

Do not guess alternate package names unless they are explicitly published later.

## Default workflow

- If auth is missing, run `jobber auth import-agent-browser`
- Sanity check auth: `jobber doctor --json`
- Check current identity: `jobber whoami --json`
- Browse operation inventory: `jobber operations list --search client --json`
- Inspect one operation: `jobber operations inspect ClientDefaults --json`
- Run raw GraphQL when needed: `jobber graphql run --operation-name CurrentAccount --query 'query CurrentAccount { account { id name inTrial industry } }' --json`

## Auth

Preferred from a live logged-in browser session:

- `jobber auth import-agent-browser`

Other supported paths:

- `printf '%s' "$JOBBER_COOKIE_HEADER" | jobber auth set-cookie-header --stdin`
- `JOBBER_COOKIE_HEADER=... jobber doctor --json`

Avoid pasting full cookie headers into logs or chat.
`jobber auth import-agent-browser` assumes a live authenticated `agent-browser` session or attached browser on the same machine.

## Constraints

- This is a private-surface adapter and is therefore fragile by definition.
- Prefer read operations before mutations.
- `operations list` is a discovered inventory, not a guarantee that every operation has a recovered query document.
- Use `graphql run` to prototype unsupported workflows before adding typed commands.

---
> Source: [danielgwilson/jobber-cli](https://github.com/danielgwilson/jobber-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
